---
title: "Beyond CVE Scanning: The Case for a Hardened Container Image Catalog"
date: 2025-11-29T10:00:00Z
tags: [
  "devsecops", "security", "supply-chain-security", "containers", 
  "docker", "kubernetes", "dora", "nis2", "chainguard"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Why traditional vulnerability scanning isn't enough and how a hardened image catalog is essential for modern enterprise security and regulatory compliance."
canonicalURL: "https://www.msbiro.net/posts/the-case-for-hardened-container-image-catalogs/"
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
    alt: "A shield protecting a chain of code blocks"
    caption: "Hardened images are a critical link in the software supply chain"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In my last few years as a Team Leader DevSecOps, I've spent a significant amount of time helping customers, mostly in the financial sector, navigate the complexities of cloud-native security. I have seen companies invest heavily in state-of-the-art runtime protection, CNAPPs, and sophisticated CI/CD security gates. Yet, a familiar pattern emerges time and again: the moment security teams start looking at vulnerability reports, chaos ensues. The numbers are just too high to handle, creating a paralyzing sense of alert fatigue.

This isn't a theoretical problem. We see it in real-world events, like the recent retirement of the community-maintained Ingress-NGINX project, which left countless users scrambling to migrate away from a critical piece of infrastructure that would no longer receive security patches. It’s a stark reminder that unmanaged dependencies are ticking time bombs.

---

### The Sisyphean Task of Reactive Security

The default reaction to this flood of vulnerabilities is often to double down on existing measures. "We need more security checks in the CI/CD pipeline," teams will say. "Let's improve the quality of our scanners or add another tool." While these steps are valuable and necessary, they are fundamentally reactive. Improving detection and shifting left helps, but it’s rarely enough to solve the problem at its source. You're still trying to clean up a mess that is being continuously created.

The root of the problem lies in the foundation of our applications: the base container images.

---

### Regulatory Pressure is Forcing a Change

This isn't just a matter of good practice anymore; it's a mandate. New regulations in Europe, like the **Digital Operational Resilience Act (DORA)** and the **Network and Information Security Directive 2 (NIS2)**, are explicitly enforcing stricter controls over the software supply chain and vulnerability management.

- **DORA**, which targets the financial sector, requires firms to have a comprehensive ICT risk management framework. This includes continuous monitoring of the software supply chain, management of third-party risks, and the ability to demonstrate resilience. The use of Software Bill of Materials (SBOMs) is becoming a key compliance artifact to meet these demands.
- **NIS2**, which applies to a broader range of critical infrastructure, mandates that organizations address cybersecurity weaknesses throughout their supply chain. It requires assessing the security practices of suppliers and implementing measures to manage those risks effectively.

Across the Atlantic, the story is similar. In the **United States**, the White House Executive Order on Improving the Nation's Cybersecurity (EO 14028) has been a major catalyst, pushing for more secure software development practices, including the use of SBOMs. Regulatory bodies like the SEC and NYDFS are also enforcing stricter rules on cybersecurity risk management, making it clear that accountability extends to the entire supply chain.

Simply put, regulators are no longer accepting "we scanned it" as a sufficient answer. They demand proactive control and demonstrable resilience.

---

### The Real Culprit: The Wild West of Public Registries

The core issue is that developers are often free to pull any image they want from public OCI registries like Docker Hub. In the rush to meet business needs and accelerate development, they naturally make the *fast* choice, not the *secure* one. An image that "just works" and gets their application running is the winner, regardless of its contents.

This convenience comes at a steep price. Public images are often bloated with unnecessary tools, libraries, and shells, dramatically expanding the attack surface. For example, a standard `ubuntu:22.04` image pulled from Docker Hub can easily contain **over 150 known vulnerabilities** before you even add your application. In contrast, a hardened, minimal image for a language like Go or Python from a specialized provider often has **less than 5, and frequently, zero.**

![alpine Vs wolife CVEs of last week.](alpineVswolfieCve.png)
The image above from [Chainguard Academy](https://edu.chainguard.dev/chainguard/chainguard-images/vuln-comparison/wolfi-base/) shows that even Alpine, which is known and appreciated for its low footprint and CVE count, struggles when compared to a structured, well-architected image built for the zero-CVE mission. Could you imagine the confrontation with a random image from a random registry?



A better approach is not just needed, it's essential! This is where a **hardened container image catalog** comes in.

---

### Hardened Images: Build vs. Buy

A hardened image catalog is a curated collection of minimal, secure, and continuously updated base images that developers are required to use. It shifts the security model from reactive scanning to proactive prevention. In projects I've personally been involved with, a focused migration to a hardened catalog resulted in an immediate **80-95% reduction** in baseline vulnerabilities, virtually eliminating alert fatigue for developers and allowing security teams to focus on the small number of genuinely critical issues.

#### The DIY Trap

The first instinct for many organizations is to build their own. "We'll take an official image, strip it down, and patch it." While possible, doing this correctly is a monumental task.
-   **Reproducibility is Hard:** How do you ensure every build is identical and verifiable?
-   **Supply Chain Poisoning:** How can you truly trust the upstream resources you're pulling from? External repositories can be compromised.
-   **SLSA Compliance:** To do it right, you need a build process that is compliant with frameworks like **SLSA (Supply-chain Levels for Software Artifacts)**, which provides verifiable evidence of an image's provenance. 

Reaching a meaningful SLSA level (e.g., Level 3) is a significant engineering effort that requires isolated, ephemeral, and hermetic build environments.
Check the table below, taken from [official SLSA 1.2 doc](https://slsa.dev/spec/v1.2/build-requirements), to better undestand the SLSA requirements.

| Implementer     | Requirement                          | Degree       | L1 | L2 | L3 |
|-----------------|--------------------------------------|--------------|----|----|----|
| **Producer**    | [Choose an appropriate build platform](https://slsa.dev/spec/v1.2/build-requirements#producer) | ✓ | ✓  | ✓  | ✓  |
|                 | [Follow a consistent build process](https://slsa.dev/spec/v1.2/build-requirements#follow-a-consistent-build-process)      | ✓ | ✓  | ✓  | ✓  |
|                 | [Distribute provenance](https://slsa.dev/spec/v1.2/build-requirements#distribute-provenance)                             | ✓ | ✓  | ✓  | ✓  |
| **Build platform** | [Provenance generation](https://slsa.dev/spec/v1.2/build-requirements#provenance-generation) / [Exists](https://slsa.dev/spec/v1.2/build-requirements#provenance-exists) | ✓ | ✓  | ✓  | ✓  |
|                 | [Provenance generation](https://slsa.dev/spec/v1.2/build-requirements#provenance-generation) / [Authentic](https://slsa.dev/spec/v1.2/build-requirements#provenance-authentic) |  | ✓  | ✓  | ✓  |
|                 | [Provenance generation](https://slsa.dev/spec/v1.2/build-requirements#provenance-generation) / [Unforgeable](#provenance-unforgeable) |  |    |    | ✓  |
|                 | [Isolation strength](https://slsa.dev/spec/v1.2/build-requirements#isolation-strength) / [Hosted](https://slsa.dev/spec/v1.2/build-requirements#hosted) |  | ✓  | ✓  | ✓  |
|                 | [Isolation strength](https://slsa.dev/spec/v1.2/build-requirements#isolation-strength) / [Isolated](https://slsa.dev/spec/v1.2/build-requirements#isolated) |  |    |    | ✓  |




For most enterprises that aren't in the business of selling software, building and maintaining a secure, compliant, and up-to-date image catalog is simply too much work. The smarter investment is to rely on an external, specialized provider.

#### The Case for Buying a Solution

Commercial providers of hardened images have made this their core business. They offer comprehensive solutions with significant advantages:
-   **Near-Zero CVEs:** These services build images from source, including only the essential libraries and binaries. This results in "distroless" images that are incredibly small and often have zero known vulnerabilities out of the box.
-   **Rapid Remediation:** Because they control the entire build process, they can patch new vulnerabilities in hours, not weeks.
-   **Minimized Attack Surface:** The images are minimal by default, often shipping with or without a shell, drastically reducing the avenues for an attacker.

---

### Key Players in the Market

The market for hardened images is growing, but three main players stand out:

**Docker Official Images & Hardened Images:** Docker provides a set of curated official images that are a good starting point. Their newer "Docker Hardened Images" offering takes this a step further, providing images built with a SLSA Level 3 build system, continuous patching SLAs, and VEX/SBOM support. This is a strong choice for teams already heavily invested in the Docker ecosystem, offering a seamless transition.

**Minimus:** Founded by the team that created Twistlock, Minimus focuses on creating minimal, secure images with a strong emphasis on compliance. They build from their own minimal Linux distribution (MinimOS) and claim to reduce CVEs by over 95%. Their key differentiator is a deep focus on government and highly regulated sectors, with built-in conformance to standards like **FedRAMP** and support for air-gapped environments. They also integrate threat intelligence to help prioritize any remaining risks.

**Chainguard:** Chainguard has quickly emerged as a leader in this space, with a relentless focus on achieving zero-vulnerability images. They maintain their own "un-distro" called Wolfi, build everything from source, and provide SLSA-compliant builds with full SBOMs and attestations. Their catalog is extensive, covering not just applications but also common libraries and even hardened VM images. Their approach appears to be the most comprehensive, aiming to secure every layer of the stack from the ground up.

| Feature                 | Docker Hardened Images | Minimus                                   | Chainguard Images     |
| ----------------------- | :--------------------: | :---------------------------------------: | :-------------------: |
| **Build Framework**     | Debian/Alpine-based    | Proprietary (MinimOS)                     | From Source (Wolfi)   |
| **SBOMs Included**      | ✅                     | ✅                                        | ✅                    |
| **VEX Support**         | ✅                     | ⍻ (Not advertised)                        | ✅                    |
| **SLSA L3 Compliance**  | ✅                     | ⍻ (Not advertised)                        | ✅                    |
| **FedRAMP Focus**       | -                      | ✅                                        | -                     |
| **Custom Images**       | ✅                     | ✅                                        | ✅                    |
| **Key Strength**        | Docker Ecosystem       | Compliance & Air-Gapped                   | Zero-CVE Focus        |

*To see live demos or learn more, check out the official websites for [Docker](https://www.docker.com/products/hardened-images/), [Minimus](https://minimus.io/), and [Chainguard](https://www.chainguard.dev/chainguard-images).  
If I reported something wrong in the table above, don't be mad at me, drop me a DM or open a PR to this article and I will fix it!*

---

### Emerging Threats and Future-Proofing

The software supply chain is a dynamic and evolving threat landscape. The case for a hardened foundation is made even stronger by emerging, sophisticated attacks. The recent **"[npm Shai-Hulud 2.0](https://www.hackerone.com/blog/shai-hulud-2-npm-worm-supply-chain-attack)"** worm, for instance, demonstrated a new level of maturity in supply chain attacks. It didn't just steal credentials; it was self-replicating, using stolen npm keys to propagate itself to other packages maintained by a compromised developer. This highlights a critical weakness: even if your code is secure, your dependencies can be turned into weapons without your knowledge. Starting from a minimal, hardened base image reduces the number of packages and package managers that can be targeted by such attacks.

Looking forward, as new workloads like AI/ML become mainstream on Kubernetes, the need for a standardized, secure base layer will only grow. Initiatives like the **Kubernetes AI Conformance Program** aim to create a reliable foundation for running these complex workloads. A hardened catalog is essential for this, ensuring the underlying infrastructure is secure, compliant, and ready for production-grade AI.

### Putting It Into Practice: Next Steps

Adopting a hardened image catalog is the first step. Enforcing its use is the second. Here are some practical ways to do it:

**Enforce Image Provenance with Policy Engines:** Use a Kubernetes policy engine like **Kyverno** or **Gatekeeper** to create rules that only allow images to be pulled from your trusted, hardened catalog. You can write policies that validate the image registry, repository, and even check for valid signatures or attestations.

    *Example Kyverno ClusterPolicy:*
    ```yaml
    apiVersion: kyverno.io/v1
    kind: ClusterPolicy
    metadata:
      name: enforce-trusted-registries
    spec:
      validationFailureAction: Enforce
      rules:
        - name: validate-registries
          match:
            any:
            - resources:
                kinds:
                  - Pod
          validate:
            message: "Images must be pulled from our trusted hardened catalog."
            pattern:
              spec:
                containers:
                  - image: "cgr.dev/chainguard/*" | "docker.io/my-org/*"
    ```
(The snippet above is just an example for clarity. In practice, images should always be pulled from an internally approved and controlled OCI registry.)

**Monitor for Threats at Runtime:** Complement policy enforcement with runtime security monitoring. Tools like **Falco** or **Sysdig Secure** can detect suspicious behavior within running containers, such as unexpected network connections, file system modifications, or process executions (e.g., a shell being spawned in a "distroless" container). This provides a critical safety net to catch any threats that might slip through preventative controls.

---

### Conclusion

The endless cycle of scanning and patching is a battle we can't win with reactive measures alone. By shifting our focus to the software supply chain and starting with a secure foundation, we can dramatically reduce the noise and focus on the risks that truly matter. In today's cloud-native landscape, adopting a hardened image catalog isn't a luxury, it's a fundamental requirement for any enterprise that's serious about security and regulatory compliance.  
It’s time to stop chasing CVEs and start preventing them!!
