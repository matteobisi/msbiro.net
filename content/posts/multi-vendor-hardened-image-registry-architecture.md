---
title: "Beyond the Vendor Subscription: Architecting a Multi-Vendor Hardened Image Registry"
date: 2026-09-08T12:00:00+02:00
tags: [
  "container-security", "hardened-images", "supply-chain-security", "devsecops",
  "kubernetes", "dora", "nis2", "cloud-native", "architecture", "oci", "sigstore"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Buying hardened container images is only step one. Here is the architecture we built at ReeVo to ingest, validate, and mirror multi-vendor catalogs into production."
keywords: ["hardened images", "multi-vendor container registry", "OCI referrers", "container supply chain", "DORA compliance", "NIS2", "ReeVo", "image ingest architecture"]
canonicalURL: "https://www.msbiro.net/posts/multi-vendor-hardened-image-registry-architecture/"
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
    alt: "Multi-vendor hardened container image registry architecture"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In 2026, regulatory pressures like DORA, NIS2, the EU Cyber Resilience Act, alongside the rapid integration of containerized AI workloads, pushed organizations to finally solve base image vulnerability debt. Procurement signed agreements with hardened image providers, security leadership checked the compliance box, and teams celebrated slashing thousands of legacy CVEs from their vulnerability dashboards.

Then Monday morning arrived.

Platform teams quickly realized that buying a commercial subscription to a hardened image catalog is only step zero. Paying an upstream vendor does not answer the day-to-day operational questions: how do developers consume these images safely? What happens when a single vendor catalog cannot cover your legacy WebSphere servers or specialized Java runtimes? How do you prevent developers from pulling unverified public images, while still preserving build speed?

At ReeVo, we worked with several enterprise customers facing this exact transition. We designed and implemented a vendor-neutral reference architecture for an internal hardened image registry. In this post, I will share the architectural principles and operational realities you cannot afford to miss when moving from vendor subscription to real production pipelines.

---

## The Single-Vendor Illusion in the Enterprise

The initial corporate desire is almost always consolidation: pick one hardened image provider (such as Chainguard, Docker Hardened Images, Red Hat UBI, or Canonical Chisel) and mandate it across the company. 

In practice, that assumption collapses during the first sprint.

Real enterprise application portfolios are heterogeneous. Your modern microservices might thrive on a lightweight distroless or Wolfi base; your data science and AI teams require specialized CUDA and Python stacks; your legacy core banking services might depend on specific IBM WebSphere runtimes or specific builds of Eclipse Temurin Java 8 and 11. No single vendor catalog provides 100% coverage across these disparate worlds. 

If you force developers into a single-vendor straightjacket, you get friction, shadow IT, or stalled migrations. If you open the gates and let pipelines pull directly from three different external registries, you end up with chaotic token management, divergent SLAs, fractured audit trails, and unpredictable upstream rate limits.

What you actually need is an **Internal Curated OCI Registry**: a single control plane inside your security perimeter that ingests, validates, and serves hardened base images from multiple upstream providers.

---

## Architectural Principles of the Ingest Control Plane

To build a reliable ingestion engine, we established a set of core principles that govern how images enter the organization.

### 1. Gate at Ingest, Not at Runtime

Many security teams rely heavily on Kubernetes admission controllers to block non-compliant images right before pod scheduling. While admission control remains a valid defense-in-depth layer, using it as your primary security gate creates severe operational friction. Blocking a release at 2:00 AM because an upstream tag suddenly gained a vulnerability disrupts operations and causes panic.

The right place to filter risk is at the ingestion boundary. An image is inspected, scanned, validated, and signed before it ever enters the internal catalog. Once an image lands in the internal production registry, developers know it is cleared for consumption.

### 2. Immutable Digest Pinning and Anti-TOCTOU

Never trust a mutable tag across asynchronous pipeline steps. A classic Time-of-Check to Time-of-Use (TOCTOU) vulnerability occurs when a pipeline resolves and scans `python:3.12-hardened`, finds zero CVEs, and then issues a copy command for `python:3.12-hardened` five minutes later. If the upstream provider pushed an update between those two steps, you just copied unvetted code into your repository.

The ingestion engine must resolve the upstream tag to its cryptographic `sha256` digest immediately:

```text
source:tag  -->  resolve  -->  source@sha256:7f9a...
```

Every subsequent action (vulnerability scanning, policy evaluation, license checking, and copying) must explicitly target that resolved digest.

### 3. Faithful, Byte-for-Byte Mirroring

When copying artifacts from upstream vendors into your internal registry, the image layers and manifest must remain byte-for-byte identical. Re-compressing layers, modifying timestamps, or converting media types breaks upstream SLSA provenance, alters digest signatures, and invalidates upstream attestations.

---

## The Ingestion Flow in Action

The ingestion pipeline functions as an automated gatekeeper. The workflow executes deterministically through six sequential stages:

{{< mermaid >}}
graph TD
    Upstream["Upstream Vendor (Chainguard, DHI, Red Hat)"] --> Step1["1. Resolve Digest (Anti-TOCTOU)"]
    Step1 --> Step2{"2. Idempotency Check"}
    Step2 -- "Already in Catalog" --> DoneSkip["Skip (No-op)"]
    Step2 -- "New Digest" --> Step3["3. Multi-Arch Blocking Scan"]
    Step3 -- "Policy Failed" --> Reject["Reject and Alert"]
    Step3 -- "Policy Passed" --> Step4["4. SBOM and License Audit"]
    Step4 --> Step5["5. Graph-Verbatim Copy"]
    Step5 --> Step6{"6. Post-Copy Verification"}
    Step6 -- "Digest Mismatch" --> FailDigest["Delete and Quarantine"]
    Step6 -- "Digest Match" --> Step7["7. Internal Cosign Signing and Catalog Release"]
{{< /mermaid >}}

Let us break down the critical technical challenges within this flow.

---

## The Copy Engine Trap: Referrers and Multi-Arch

The copy engine is one of the most deceptively complex components in the entire architecture. Teams often assume that a generic container CLI or a Docker promotion plugin is sufficient. In production, three major pitfalls appear:

### The OCI Referrers Graph

Modern hardened images do not travel alone; they arrive with a graph of associated OCI artifacts attached via the OCI 1.1 referrers API. This graph contains cryptographic Cosign signatures, SPDX or CycloneDX SBOMs, and in-toto provenance attestations. 

If your copy engine performs a naive image copy, it leaves the entire referrers graph behind in the upstream registry. Your internal registry receives the image layers, but loses the cryptographic proof of authenticity and the supply-chain bills of materials. Your copy tool must be OCI referrers-aware, recursively copying the root manifest index, the per-platform manifests, and the complete referrer tree.

### Docker Manifest Lists vs OCI Indexes

Different registries and vendors mix OCI image specs and legacy Docker media types (`application/vnd.docker.distribution.manifest.list.v2+json` versus `application/vnd.oci.image.index.v1+json`). A copy tool that inadvertently converts media types during transfer will alter the manifest digest, causing post-copy verification to fail. You need a tool capable of graph-verbatim copying across registry boundaries.

### Post-Copy Verification

Never assume a network copy succeeded simply because the HTTP status code was 200. Certain enterprise registries normalize manifests or recalculate hashes upon ingestion. The pipeline must re-resolve the target catalog digest immediately after copying and verify equality:

```bash
SOURCE_DIGEST=$(skopeo inspect --raw docker://upstream.vendor.com/app@sha256:abc... | sha256sum)
TARGET_DIGEST=$(skopeo inspect --raw docker://registry.internal.corp/catalog/app@sha256:abc... | sha256sum)

if [ "$SOURCE_DIGEST" != "$TARGET_DIGEST" ]; then
  echo "CRITICAL: Digest mismatch after copy. Rejecting catalog entry."
  exit 1
fi
```

---

## The Illusion of the Zero-CVE Scan

Hardened images are celebrated for achieving zero known CVEs. However, when you operate an ingest pipeline, a scan returning zero vulnerabilities is not always proof of perfection; it can also indicate a broken scanner.

In our field implementations, we frequently encounter three false-clean scenarios:

1. **Distro Lag and Missing Scanner Feeds**: When vendors release images based on brand-new distributions (such as minimal undistro roots or fresh OS releases), public vulnerability databases often suffer a multi-week lag. A version-naive scanner that does not recognize the OS reports zero vulnerabilities simply because it has no reference data.
2. **Backporting Falsely Flags or Misses**: Enterprise distributions (Red Hat, Ubuntu Pro) backport security patches into older upstream package versions without bumping major version numbers. Scanners that fail to parse vendor-specific OVAL or security advisories produce massive false positive noise, or worse, miss actual gaps.
3. **The Silent Failure Trap**: If a scanner CLI fails to download its daily vulnerability database and falls back to a stale or empty local cache, scans finish in milliseconds with zero reported issues.

To prevent blind spots, implement a **control test** in your CI pipeline: periodically scan a known-vulnerable canary image alongside your catalog targets. If the canary image reports zero CVEs, abort the pipeline immediately; your scanning engine is compromised.

Furthermore, employ a **two-pass scanning strategy**: pass one generates a complete, non-blocking software inventory and vulnerability report for audit records; pass two enforces a strict, blocking gate on high and critical exploitable CVEs.

---

## Lifecycle Operations: Tagging and Continuous Re-Scan

Ingesting an image is not a one-time transaction. A container image that is completely clean on Tuesday may be subject to a Critical Zero-Day by Friday. An internal registry must support ongoing lifecycle management.

### Dated Snapshots versus Floating Tags

Developers need simplicity; production systems need immutability. To satisfy both, publish dual tag families:

* **Floating Tags for Development**: Tags like `python-3.12:latest` or `python-3.12:stable` provide local developer ergonomics and automatic non-breaking updates in sandbox environments.
* **Dated Immutable Snapshots for Production**: Tags formatted as `python-3.12:2026.09.08` or explicit SHA256 digest pinning are required for staging and production deployments. Once published, a dated snapshot is immutable and never overwritten.

### Continuous Asynchronous Re-Scanning

Because vulnerability databases update constantly, every active image in the internal catalog must be re-scanned on a scheduled cadence (for instance, every 12 to 24 hours). 

When a new vulnerability surfaces in an existing catalog image, the alerting engine should not generate noisy broadcast spam. Notifications must route dynamically based on ownership: platform engineers receive alerts for base OS layer defects, while application teams receive alerts when custom application layers are affected.

---

## Meeting DORA and NIS2 in Production

Setting up this architecture directly addresses European regulatory demands without creating parallel compliance paperwork:

* **DORA (Articles 5 to 16, Art. 10 Detection, and Third-Party ICT Risk)**: Demonstrates that external container software dependencies are continuously verified, monitored against SLAs, and inventoried through immutable digests and re-scans.
* **NIS2 (Article 21 Supply Chain Security and Cyber Hygiene)**: Satisfies supply chain validation obligations by maintaining a centralized asset inventory, cryptographically verified provenance, and automated vulnerability disclosure workflows.
* **CRA (Vulnerability Handling and SBOM Mandates)**: Ensures that every base image distributed downstream carries verifiable machine-readable SBOMs and documented patch lifecycles.

---

## Summary and Next Steps

Buying a hardened image subscription solves the upstream build challenge, but the downstream responsibility remains entirely yours. 

Building an internal ingestion control plane allows you to source images from multiple specialized vendors, maintain strict digest equality, preserve OCI referrers, and provide development teams with trusted, frictionless building blocks.

In the next article, we will examine the pipeline code and configuration files, detailing how to implement this workflow using open-source tools such as Skopeo, Trivy, Cosign, and GitHub Actions.