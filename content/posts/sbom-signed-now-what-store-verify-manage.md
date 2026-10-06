---
title: "You Built and Signed Your SBOM. Now What? Store, Verify, Manage"
date: 2026-10-06T06:50:00+02:00
tags: [
  "cybersecurity", "supply-chain-security", "SBOM",
  "open-source", "cloud-native", "devsecops", "sigstore", "kubernetes"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "SBOMs fail when we abandon them after signing. Where to store your SBOM, how to verify it is real, and how to manage retention and continuous scanning with FOSS tooling."
canonicalURL: "https://www.msbiro.net/posts/sbom-signed-now-what-store-verify-manage/"
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "SBOM lifecycle after signing"
    caption: "Signing is the middle, not the end"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Javier Martinez published a sharp piece on September 24, 2026: [Why are SBOMs failing to stop supply chain attacks?](https://www.sysdig.com/blog/why-are-sboms-failing-to-stop-supply-chain-attacks) I agree with his analysis. I recommend reading it before continuing here.

## TLDR of the Sysdig Article

Martinez starts from a simple idea. An SBOM plus signatures and attestations should give us attribution (who built it), provenance (where it comes from) and content (what is inside). In a Kubernetes flow that means verifying attestations before deploy, then generating and signing your own SBOM when you ship.

On paper it closes most supply chain vectors. In practice it does not, for four reasons:

1. Inconsistent tool support. Signatures and attestations work, but wiring them across developers, CI and clusters is painful. Low adoption drives low tool investment, and that keeps adoption low. Apple notarization is cited as the counterexample where tooling makes signing almost invisible.
2. You cannot trust every SBOM. An ingredient list does not prove the absence of Salmonella. Producer SBOMs are useful for a first look, but registries rarely rescan and consumers must run their own scanners in CI and at admission.
3. Vulnerabilities change over time. A scan is a timestamp. New CVEs land daily, so an SBOM that embeds a vulnerability list goes stale fast unless a CNAPP keeps the SBOM stored and re-evaluates it against fresh intelligence without rescanning the image.
4. Internal threats. XZ Utils in 2024 showed the limit. A trusted insider can ship malicious code that no static scanner catches. You need execution analysis, code review discipline and, carefully applied, AI assisted detection in the pipeline.

His conclusion is pragmatic. SBOMs remain the standard interchange format between CNAPP components, and every link in the chain should check and attest. He frames this as voluntary and likely in need of regulation to stick, which reflects the US context. In the EU that regulation already exists in the CRA. It requires manufacturers to create a machine readable SBOM for at least top level dependencies, keep it in the technical documentation and provide it to market surveillance on request. Start inside your own perimeter now, because for products placed on the EU market this is becoming a conformity requirement, not an option.

I agree with all of it. The part I think is missing is what happens the minute after you did everything right. You generated the SBOM at the right time, you signed it. What now? In my experience this is where most programs die. The SBOM sits in a CI artifact bucket nobody queries, nobody verifies the signature at deploy time, nobody rescans it six months later when OpenSSL burns again.

This post covers that missing operational half with FOSS tooling and practices I would actually run.

---

## Generate at the Right Moment, Then Sign

If the SBOM is generated at the wrong time it is fiction on day one.

Best practice in cloud native pipelines is to generate the SBOM from the final artifact you ship, not from a source manifest collected three steps earlier. For containers that means after the image build, from the image filesystem and image config, including the base image layers, OS packages and language packages. Tools like [Syft](https://github.com/anchore/syft), [Trivy](https://github.com/aquasec/trivy) (`trivy sbom`), [Docker Buildx with attestations](https://docs.docker.com/build/metadata/attestations/sbom/) and [apko/ko/melange](https://github.com/chainguard-dev/apko) for distroless style builds all support this pattern. Generate in SPDX or CycloneDX, both NTIA compliant, and prefer CycloneDX if your consumer is [OWASP Dependency-Track](https://dependencytrack.org/) and SPDX if your consumer leans toward license compliance workflows. [protobom](https://github.com/protobom/protobom) and [bomctl](https://github.com/bomctl/bomctl) help when you need to convert or merge formats without losing fields.

Signing comes immediately after, in the same trusted job. The standard stack today is [Sigstore](https://www.sigstore.dev/):

- [Cosign](https://github.com/sigstore/cosign) to sign the image and attach the SBOM and SLSA provenance as attestations
- [Fulcio](https://github.com/sigstore/fulcio) for short lived certificates via OIDC (no long lived keys in CI)
- [Rekor](https://github.com/sigstore/rekor) as transparency log so anyone can later prove when that signature existed
- [in-toto](https://in-toto.io/) attestations and [SLSA provenance](https://slsa.dev/) to bind source, builder, build steps and materials to the output

A minimal hardened flow looks like this:

```bash
# build with provenance + SBOM attestation
docker buildx build --push \
  --provenance=true --sbom=true \
  -t ghcr.io/acme/api:v1.4.2 .

# generate a portable CycloneDX copy for the inventory
syft ghcr.io/acme/api:v1.4.2 -o cyclonedx-json > sbom.cyclonedx.json

# sign image + attest SBOM with keyless Sigstore
cosign sign --yes ghcr.io/acme/api:v1.4.2
cosign attest --yes --predicate sbom.cyclonedx.json \
  --type cyclonedx \
  --predicate-type https://cyclonedx.org/specification/overview/ \
  ghcr.io/acme/api:v1.4.2
```

Pin your GitHub Actions to SHA, set `permissions: {}` by default and grant `id-token: write` only to the sign job. I detailed that hardening in [Supply Chain Attacks Won't Stop: 8 Controls](https://www.msbiro.net/posts/supply-chain-attack-prevention-8-controls/). Without it your signing step attests attacker controlled content.

---

## Where to Store the SBOM

This is the first question teams get wrong. An SBOM stored where nobody can find it during an incident is equivalent to no SBOM.

You need two storage planes, and they serve different queries.

### 1. With the artifact, for deploy time verification

Since OCI image spec v1.1, registries support the [referrers API](https://specs.opencontainers.org/image-spec/docs-spec-guidance/#referrers). Cosign signatures, SBOM attestations and SLSA provenance travel as referrers bound to the image digest, not the mutable tag. When you pull `sha256:abc...` you can discover `sha256:abc...` has SBOM X and signature Y.

Use a registry that actually implements this:

- [Harbor](https://goharbor.io/) (native SBOM view, scheduled rescan with Trivy, immutable tags, proxy cache)
- GHCR, Docker Hub, ECR, GAR, ACR (referrers support varies by version, test `cosign verify` and `docker scout attest` against your instance)
- [Zot](https://zotregistry.dev/) if you want a lightweight OCI native FOSS registry

Harbor deserves a special mention for platform teams. It keeps the SBOM next to the image, shows OS and language CVEs per tag, and can block pull or replication on policy. That covers a large part of Martinez challenge 2 without building custom glue.

### 2. In an inventory, for search over time

Referrers answer "is this image valid right now". They do not answer "which of my 800 services ships Log4j 2.14" at 2 AM. For that you need an SBOM repository that indexes every build.

FOSS options I would shortlist:

- [OWASP Dependency-Track](https://dependencytrack.org/): upload every CI SBOM via API, it correlates with NVD, GitHub Advisories and OSV, tracks VEX, and keeps historical portfolio views per project version
- [GUAC (Graph for Understanding Artifact Composition)](https://guac.sh/) (OpenSSF, includes [Trustify](https://github.com/guacsec/trustify) contributed by Red Hat, formerly Trustification): ingests SBOMs plus SLSA, VEX and Scorecard data into a graph so you can traverse "image depends on package affected by CVE", with Trustify as searchable backend for SBOM and advisory metadata with API first design
- Plain immutable object storage (S3 with Object Lock, GCS) as legal archive behind the above, keyed by `artifact digest / sbom format version / generator version`

Push on every build, not only on release. Tag each upload with project, version, git SHA, image digest, builder identity and pipeline URL. Without those fields your SBOM lake becomes unqueryable in weeks.

---

## Who Assures You the SBOM Is Real

Nobody should take a producer SBOM at face value. Martinez is right, and this is where verification tooling closes the gap.

Think in three checks, applied at different gates.

First, signature and provenance verification. At admission time, verify Cosign signature, Rekor inclusion and SLSA provenance before the pod starts. In Kubernetes the FOSS path is:

- [Ratify](https://github.com/deislabs/ratify) as verification engine on the cluster
- [Kyverno](https://kyverno.io/) or [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) to deny images without valid attestations
- [Sigstore policy-controller](https://github.com/sigstore/policy-controller) if you are already on Sigstore native policy

Example Kyverno intent in plain language: allow only `ghcr.io/acme/*` digests signed by our CI OIDC identity with a CycloneDX attestation present. Anything else stays Pending.

Second, independent regeneration. For critical base images and golden paths, rebuild the SBOM yourself with Syft or Trivy in your pipeline and diff it against the vendor SBOM using `bomctl diff` or Dependency-Track upload in a staging project. Drift means either tool divergence (common across ecosystems) or tampering (rare but high signal). I run this check on base image updates and on every third party image promotion from `quarantine` to `trusted` in Harbor.

Third, transparency and revocation. Verify Rekor entry timestamp, check Fulcio issuer matches your builder (for example `https://token.actions.githubusercontent.com` with your repo identity), and enforce expiry. Short lived certs reduce key theft windows, but you still need a revocation story for compromised builders. That is rotation of builder identity, blocking old digests at admission, and publishing a VEX statement that marks affected artifacts.

Projects to watch here are [gittuf](https://gittuf.dev/) for moving trust out of the forge into the repository itself, and [SBOMit](https://github.com/sbomit/sbomit) for adding verification layers to SBOM distribution. I mentioned gittuf in my 8 controls post because Trusted Publishing abuse (Bitwarden CLI, March 2026) showed the forge alone cannot be the root of trust.

---

## How to Manage It: Retention, Rescan, Response

An SBOM is a living record. If you treat it as a PDF attached to a release ticket, you repeat Martinez challenge 3.

### Retention aligned to risk, not disk space

Keep the SBOM at least as long as the artifact is supported or deployed, plus your legal hold. Practical anchors:

- Internal services: retain SBOM for image digest lifetime plus 12 months after last deployment, so post mortems can reconstruct exposure
- Regulated or sold software: the EU Cyber Resilience Act requires technical documentation retention for 10 years after placing on the market, with SBOMs in that evidence bundle. In the US, EO 14028 and CISA guidance push federal suppliers toward machine readable SBOM delivery and storage
- Immutable archive: WORM storage with hash chaining, so auditors and customers can verify you did not rewrite history after a CVE

Dependency-Track keeps project versions forever by default. Define an archival policy (for example, keep every release SBOM, purge per commit CI SBOMs after 180 days) or your database and NVD sync jobs will slow down.

### Automatic rescan over time

Do not rescan images by rebuilding them nightly. Re-evaluate stored SBOMs against fresh vulnerability feeds. That is exactly the CNAPP pattern Martinez describes, and you can run it with FOSS:

- Dependency-Track scheduled sync with NVD, OSV and GitHub Advisories plus internal analyzer cadence (hourly to daily)
- Harbor scheduled vulnerability rescan per project when Trivy DB updates
- GUAC polling OSV and ingesting new CSAF/OpenVEX statements to update the graph without touching images
- VEX automation with [OpenVEX](https://github.com/openvex/openvex) (`vexctl`): publish `not_affected` with justification or `affected` with remediation when your team triages, consume upstream VEX from vendors to suppress noise

Prioritize with exploitability, not CVSS alone. Feed EPSS scores and CISA KEV catalog into triage, block deploy on KEV or reachable critical, create tickets for the rest. Scanners flag, VEX explains, policy decides.

### Incident response is the payoff

When the next OpenSSL or Axios style advisory drops, your runbook should be three queries, all answered from inventory, not from rebuilding the world:

- Which SBOMs contain package P at version range V
- Which of those map to running images in which clusters and namespaces
- Which of those are actually exploitable per VEX and runtime reachability

GUAC excels at the first traversal. Dependency-Track portfolio search plus Harbor artifact search plus a simple cluster image inventory (`kubectl` image list exported nightly) covers the same with less graph machinery. Either way, test it with a fire drill before you need it. If answering takes more than an hour, your storage model is wrong.

Remediation closes the loop. [Dependabot](https://github.com/dependabot) or [Renovate](https://github.com/renovatebot/renovate) open the bump PR, CI regenerates and resigns the SBOM, admission verifies the new digest, inventory marks the old digest deprecated. The SBOM diff becomes part of the change review, alongside code diff.

---

## A Minimal FOSS Stack That Works

If I were starting a platform team tomorrow with zero budget and CRA style duties, I would run this and iterate, since it covers generation, storage, verification and continuous monitoring without proprietary lock in.

- Generation: Syft plus Grype or Trivy in CI, Docker Buildx SBOM attestations for images, protobom or bomctl for format normalization
- Signing: Cosign keyless with Fulcio and Rekor, SLSA provenance via [slsa-github-generator](https://github.com/slsa-framework/slsa-github-generator)
- Store with artifact: Harbor with immutable tags and Trivy rescan enabled
- Inventory and monitor: Dependency-Track as system of record, GUAC when dependency graph queries outgrow portfolio lists
- Verify at deploy: Ratify plus Kyverno deny rules, Sigstore policy-controller where Sigstore native fits
- Triage and communicate: OpenVEX statements published next to releases, CSAF for enterprise customers if required
- Remediate: Renovate plus image promotion pipeline from quarantine to trusted

This is intentionally boring technology. Every component is maintained, documented and already integrated with the others. The hard part is process discipline, not tool choice.

---

## Closing Thought

Martinez ends with a call for regulation, and for a US audience that makes sense. For readers in the EU, that future is already scheduled. The CRA makes SBOM generation, vulnerability handling and 10 year retention a legal duty, with reporting duties applying since September 2026 and full applicability from December 2027. NIS2 and DORA add indirect pressure through supply chain and third party risk duties, even if they do not mandate SBOMs by name. Regulation will therefore not only broaden generation, it will set the baseline for tool defaults and enforcement.

Even with that baseline, the operational gap is ours to close. A signed SBOM nobody stores, verifies and re-evaluates is compliance theater. A signed SBOM stored with the artifact, mirrored in a searchable inventory, verified at admission and rescanned daily becomes what it was meant to be, a trust anchor you can query during an incident instead of a file you attach to a ticket.

Build it at the right time, sign it with a short lived identity, keep it where both the cluster and the analyst can find it, assume it is guilty until verification says otherwise, and plan to live with it for years. That discipline turns SBOMs from paperwork into detection and response leverage.

---

## References

- Javier Martinez, Sysdig: [Why are SBOMs failing to stop supply chain attacks?](https://www.sysdig.com/blog/why-are-sboms-failing-to-stop-supply-chain-attacks)
- Sigstore docs: [Cosign, Fulcio, Rekor, policy-controller](https://docs.sigstore.dev/)
- OCI spec: [Referrers API](https://specs.opencontainers.org/image-spec/docs-spec-guidance/#referrers)
- FOSS inventory: [Dependency-Track](https://dependencytrack.org/), [GUAC including Trustify](https://guac.sh/), [Harbor](https://goharbor.io/)
- Verification on Kubernetes: [Ratify](https://github.com/deislabs/ratify), [Kyverno](https://kyverno.io/), [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)
- VEX and exchange: [OpenVEX spec and vexctl](https://github.com/openvex/openvex), [bomctl](https://github.com/bomctl/bomctl), [protobom](https://github.com/protobom/protobom)
- Related on this blog: [Supply Chain Attacks Won't Stop: 8 Controls](https://www.msbiro.net/posts/supply-chain-attack-prevention-8-controls/)
