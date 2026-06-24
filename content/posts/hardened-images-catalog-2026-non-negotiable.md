---
title: "In 2026 I Am Still Asked Why You Need a Hardened Container Image Catalog"
date: 2026-06-24T09:30:00+01:00
tags: [
  "supply-chain-security", "hardened-images", "containers", "devsecops",
  "dora", "nis2", "compliance", "kubernetes", "container-security",
  "cybersecurity", "cloud-native", "distroless", "alpine", "sbom", "slsa"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Why hardened container image catalogs are non-negotiable in 2026: the technological case, the DORA mandate, and the NIS2 obligations explained."
images:
 - "https://www.msbiro.net/social-image.png"
canonicalURL: "https://www.msbiro.net/posts/hardened-images-catalog-2026-non-negotiable/"
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
    alt: "Shield protecting container images in a supply chain"
    caption: "Hardened image catalogs are not optional in 2026"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

It's 2026 and I still receive questions from customers and colleagues about why they should adopt a hardened container image catalog, why it matters, and how to justify the investment internally. I hear it from security engineers, from architects, from technical leads at companies that are otherwise doing serious work on their security posture.

The honest answer is short: European regulations like DORA and NIS2 require it, and from a purely technological standpoint it is the logical evolution of how we have always managed infrastructure. Both arguments stand independently. Together they leave no room for debate.

Let me explain each of them.

---

## The Technological Case: Who Owns the Base Image?

Before containers, before Kubernetes, before the DevOps movement reshaped how software gets shipped, corporations operated under a clear model. System engineers were responsible for the full infrastructure stack. They selected the operating system, installed and hardened the application servers, configured the databases, managed patching schedules, and maintained a documented lifecycle for every component. Developers received a secured environment and focused on deploying their application. The responsibility boundary was explicit and enforced.

Then the container revolution arrived. Developers gained something genuinely powerful: the ability to define the entire execution environment in a Dockerfile, from the OS layer to the runtime, packaged and portable. That capability accelerated delivery enormously and the industry rightly embraced it. The problem is that many organizations adopted the speed of containers without ever answering a foundational question: who is now responsible for the base image selection and its lifecycle?

In practice, across many non-regulated environments, the answer quietly became "whoever writes the Dockerfile." That means developers are now making decisions that, in the pre-container world, belonged to specialized systems engineers with security training and formal accountability. Not because developers are irresponsible, but because no organizational policy was ever set. The result is entirely predictable. Base images get chosen by habit, by what worked on someone's laptop, by what was familiar during a previous job, or by what made debugging easier at 11pm before a release. A full Ubuntu image with a shell and 200 packages does run the same application as a minimal distroless image. The developer does not experience the difference in CVEs. The security team does, every week, on every scan report.

No enterprise would allow developers to freely choose which operating system version to install on production servers, or skip the hardening checklist because it slowed things down. Those assets had a defined lifecycle, an assigned owner, and an approval process. Container base images are infrastructure. They deserve the same governance that was always applied to the operating systems and application servers they replaced. The principle has not changed; only the packaging has.

---

### Alpine vs Hardened Images: Understanding the Gap

A question I hear often is: "My developers told me we are fine because we use Alpine. Is that true?"

Alpine is a genuine improvement over pulling an unvetted Debian or Ubuntu image from a public registry. It is small, it has fewer packages than a general-purpose distribution, and its CVE count is typically lower. If your baseline was a full distro image, moving to Alpine is a step in the right direction.

It is not, however, the same as using a hardened or distroless image, and the gap matters.

Alpine still ships with a shell (ash), a package manager (apk), and a collection of BusyBox utilities. Each of those is not just a convenience for the developer: it is also a potential tool for an attacker who achieves code execution inside the container. In a properly distroless image there is nothing to use. No shell, no package manager, no wget, no curl. The container runs the application runtime and nothing else. An attacker inside a distroless container has a dramatically reduced set of options for moving laterally or escalating privileges.

The CVE picture is also meaningfully different. Alpine reduces the vulnerability count compared to a standard distribution, but it regularly carries tens of CVEs across its packages at any given time. Hardened images built from source, using minimal distributions like Wolfi, consistently achieve zero CVEs out of the box and patch new vulnerabilities within hours of disclosure, backed by a contractual SLA. The difference is not marginal; it is structural.

There is a third dimension that Alpine cannot provide: auditability. Hardened image providers ship each image with a Software Bill of Materials (SBOM), a VEX document that contextualizes any reported vulnerability for that specific build, and build provenance attestations that prove the image was produced in a controlled, isolated environment reaching SLSA Build Level 3. When a regulator or auditor asks you to demonstrate the integrity of your software supply chain, those artifacts are concrete, verifiable evidence. Alpine does not ship any of them.

The table below summarizes where the two approaches differ:

| Dimension | Alpine | Hardened / Distroless |
| :-- | :-- | :-- |
| **Shell present** | Yes (ash / BusyBox) | No |
| **Package manager present** | Yes (apk) | No |
| **Typical base CVE count** | 10 to 50+ | 0 to 5 |
| **Patch SLA** | Community cadence | Hours (vendor contractual) |
| **SBOM included** | No | Yes |
| **Build provenance (SLSA L3)** | No | Yes (commercial providers) |
| **VEX support** | No | Yes |

Alpine moved the ecosystem in the right direction. Distroless and hardened images finish the job.

---

## The Regulatory Case: DORA and NIS2 Remove the Choice

If the technological argument alone is not sufficient to drive a decision for your organization, the regulatory argument is. And for a large portion of European enterprises, it is no longer a question of best practice but of legal obligation.

I covered the DORA angle in detail in a [previous post on this blog](/posts/secrets-cnapp-0cve-dora), which I recommend reading for the full context on how hardened images connect to the broader DORA compliance picture alongside secrets management and CNAPP platforms. The short version: the [Digital Operational Resilience Act (Regulation EU 2022/2554)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022R2554) has been enforceable since January 17, 2025, and its requirements on ICT risk management, vulnerability management, and third-party supply chain oversight are explicit. A governed, hardened container image catalog is one of the most direct and auditable implementations of those requirements.

What has changed in 2026 is the scope. [NIS2 (Directive EU 2022/2555)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022L2555) extends the same logic to a much wider population of organizations, covering essential and important entities across critical sectors including energy, transport, banking, health, digital infrastructure, manufacturing, and public administration. If your organization falls under NIS2, supply chain security is not a recommendation; it is a legal obligation with real enforcement consequences.

### DORA: Regulation (EU) 2022/2554

| DORA Article | Requirement | How a Hardened Image Catalog Addresses It |
| :-- | :-- | :-- |
| **Article 6: ICT Risk Management Framework** | Establish and maintain a comprehensive framework to protect ICT assets from risks including vulnerabilities and unauthorized access. | A curated catalog with a documented lifecycle reduces the attack surface at the base image layer and provides an auditable, governed foundation for the ICT risk framework. |
| **Article 9: Protection and Prevention** | Implement policies and tools to maintain confidentiality, integrity, and availability. Address technical vulnerabilities proactively. | Images with zero or near-zero CVEs, continuous vendor-backed patching, and verified provenance directly satisfy the requirement to proactively address technical vulnerabilities. |
| **Article 10: Detection and Patch Management** | Track, prioritize, and evidence remediation of vulnerabilities in ICT systems including containers and images. | Hardened image providers deliver patch SLAs measured in hours, with SBOMs and VEX documents that provide the audit trail required to evidence remediation. |
| **Article 28: ICT Third-Party Risk Management** | Perform due diligence on third-party ICT providers and enforce appropriate security standards contractually. | Selecting a certified hardened image provider with SLSA L3 provenance, SBOMs, and contractual security SLAs is a concrete and demonstrable act of third-party supply chain risk management. |

### NIS2: Directive (EU) 2022/2555

| NIS2 Article | Requirement | How a Hardened Image Catalog Addresses It |
| :-- | :-- | :-- |
| **Article 21(2)(a): Risk Analysis and Security Policies** | Implement policies on risk analysis and information system security, proportionate to risk exposure. | A formally governed base image catalog is a documented risk control that reduces vulnerability exposure across all containerized workloads and fits directly into a security policy framework. |
| **Article 21(2)(d): Supply Chain Security** | Address security-related aspects of relationships with direct suppliers and service providers. | Selecting a hardened image supplier with documented security practices, build provenance, and contractual vulnerability SLAs is a direct implementation of supply chain security obligations. |
| **Article 21(2)(e): Security in Systems Acquisition and Maintenance** | Address vulnerability handling and disclosure across the acquisition and maintenance lifecycle of network and information systems. | A base image catalog with an active lifecycle policy, automated CVE monitoring, and vendor patch notification directly satisfies the acquisition and maintenance security mandate. |
| **Article 21(2)(h): Cryptography and Encryption Policies** | Implement policies on the use of cryptography where appropriate. | Hardened image providers ship cryptographic signatures and attestations with every image, enabling organizations to enforce signature verification as part of their cryptographic controls. |

The combined scope of DORA and NIS2 covers the overwhelming majority of enterprises that operate critical or regulated services in the European Union. If your organization falls under either regulation, a hardened image catalog is not a nice-to-have; it is a compliance requirement that can be directly audited.

---

## Conclusion

In 2026 the market offers mature and well-supported solutions for this problem, and some of them are accessible at no cost. [Docker Hardened Images](https://www.docker.com/products/hardened-images/) are open source and free to use, with enterprise tiers available for SLAs and compliance certifications. [Minimus](https://minimus.io/) offers free entry-level access. [Chainguard](https://www.chainguard.dev/) provides free community images for a broad catalog of common applications. I covered the Docker announcement in more detail in a [dedicated post](/posts/docker-hardened-images-free) if you want to explore that option further.

If you are approaching this topic for the first time, the [case for a hardened container image catalog](/posts/the-case-for-hardened-container-image-catalogs) and the [practical guide to adopting distroless images](/posts/from-dev-to-prod-making-distroless-images-your-default) on this blog are good starting points before diving into vendor selection.

The budget objection no longer holds. The technical objection, as the Alpine comparison showed, does not hold either. The regulatory objection, for anyone operating under DORA or NIS2, was never available in the first place.

The conversation about whether to adopt hardened container image catalogs should be over in 2026. What remains is execution: inventorying the base images in use, identifying the highest-CVE targets, selecting a provider, and enforcing catalog adoption through policy engines like Kyverno or Gatekeeper. That work is not trivial, but it is tractable. Start with the three images that carry the most CVEs and measure the difference. The results tend to make the rest of the conversation very easy.

---
