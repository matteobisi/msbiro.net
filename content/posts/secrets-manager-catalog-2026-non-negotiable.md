---
title: "In 2026 I Am Still Asked Why You Need a Centralized Secrets Manager"
date: 2026-07-17T06:30:00+01:00
tags: [
  "secrets-management", "secrets-manager", "devsecops", "openbao",
  "hashicorp-vault", "dora", "nis2", "compliance", "kubernetes",
  "cybersecurity", "cloud-native", "git-crypt", "sops", "audit-logging", "cncf"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Why a centralized secrets manager is non-negotiable in 2026: the operational limits of git-crypt and sealed-secrets style tools, the DORA and NIS2 mandate, and why OpenBao is now the open source secrets manager to adopt."
images:
 - "https://www.msbiro.net/social-image.png"
canonicalURL: "https://www.msbiro.net/posts/secrets-manager-catalog-2026-non-negotiable/"
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
    alt: "A locked vault protecting credentials and keys inside a distributed system"
    caption: "Encrypting a file in Git is not the same as managing a secret"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

It's 2026 and I still get the same question from customers and colleagues: we already encrypt our secrets with git-crypt (or SOPS, or sealed-secrets), why do we need a full secrets manager on top of that? I hear it from platform teams that are otherwise mature, from developers who are genuinely trying to do the right thing, and from managers who see a centralized secrets manager as one more piece of infrastructure to buy, run, and justify.

The honest answer is short: encryption at rest inside a Git repository solves confidentiality of the file and leaves the harder problems untouched. European regulations like DORA and NIS2 remove any remaining room for debate.

---

## The Technological Case: Encryption Is Not Governance

Tools like [git-crypt](https://github.com/AGWA/git-crypt), [SOPS](https://github.com/getsops/sops), and Kubernetes [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) solved a real and painful problem: they let teams commit encrypted secrets alongside code instead of pasting plaintext credentials into a repository, a Slack message, or a shared spreadsheet. For a small team, or for a narrow, well-scoped use case like encrypting a handful of values in a GitOps repo, they are a legitimate and pragmatic choice. I am not arguing against using them tactically.

The problem is that many organizations stop there and treat "the file is encrypted" as equivalent to "secrets are managed." It isn't, and the gap becomes visible the moment something goes wrong.

### A Scenario I Have Seen Play Out More Than Once

Two engineers, A and B, both clone a repository protected with git-crypt. Both have a valid GPG key added to the repository's `.git-crypt` keyring, because both needed access at some point.

Engineer A runs `git-crypt unlock` on a laptop to debug a deployment issue and, out of habit, leaves the repository unlocked on disk afterwards. Months later one of three things happens: that laptop is compromised, the repository is copied to a shared drive for a migration, or engineer A leaves the company and nobody thinks to rotate the key because nobody kept a record of who held it.

When the incident response team asks "who could have accessed this secret, and did anyone actually access it," git-crypt has no answer. Decryption happens locally, offline, with no logging and no interaction with any server. There is no event to query, no timestamp, no IP address, nothing to hand to an auditor or a SOC analyst. The only trail is the Git commit history, which tells you when a file changed, never who read the decrypted content or when.

This is not a hypothetical edge case; it is the structural design of the tool:

- **No revocation without re-encryption.** Removing a former employee's access with git-crypt means re-encrypting every protected file with a new key and redistributing it to every remaining holder. With a secrets manager, revoking one identity's access is an API call that takes effect immediately, with no impact on anyone else.
- **No dynamic secrets.** A database password stored in an encrypted file is the same password until someone manually rotates it. A secrets manager can issue a short-lived database credential on demand and revoke it automatically when the lease expires, so a leaked credential has a shelf life measured in hours instead of years.
- **No fine-grained, identity-aware access control.** git-crypt and SOPS decisions are binary: you either hold a key that decrypts the whole protected set, or you don't. There is no way to say "this service can read only its own database password" without splitting repositories or files, which quickly becomes unmanageable at scale.
- **Keys travel with people, not with policy.** Once a GPG key or an age key is distributed, the tool has no way to know where else it has been copied, whether it sits in a password manager, a CI runner's environment, or an old laptop. A secrets manager authenticates every request against an identity provider at the moment of access, not at the moment of key distribution.

Secrets management is a different discipline than file encryption, built around centralized audit logging, dynamic issuance, and revocation as a first-class operation, not an afterthought that requires a repository-wide re-key.

---

## The Regulatory Case: DORA and NIS2 Remove the Choice

If the technological argument alone does not drive a decision for your organization, the regulatory one will. For a large share of European enterprises, this is no longer best practice but legal obligation.

I covered the DORA angle for secrets management in a [previous post on this blog](/posts/secrets-cnapp-0cve-dora), alongside hardened container images and CNAPPs, which I recommend reading for the broader compliance picture. The short version: the [Digital Operational Resilience Act (Regulation EU 2022/2554)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022R2554) has been enforceable since January 17, 2025, and its ICT risk management, protection, and third-party oversight articles apply directly to how credentials, API keys, and certificates are stored and rotated.

What has changed in 2026 is the scope. [NIS2 (Directive EU 2022/2555)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022L2555) extends the same logic to a much wider population of organizations, covering essential and important entities across energy, transport, banking, health, digital infrastructure, manufacturing, and public administration. Cryptography and access control obligations under NIS2 are explicit, and an encrypted file in a Git repository with no access log does not meet them.

### DORA: Regulation (EU) 2022/2554

| DORA Article | Requirement | How a Centralized Secrets Manager Addresses It |
| :-- | :-- | :-- |
| **Article 6: ICT Risk Management Framework** | Establish and maintain a comprehensive framework to protect ICT assets from risks including vulnerabilities and unauthorized access. | A centralized secrets manager gives a single, governed inventory of every credential, key, and certificate in use, replacing scattered encrypted files with no shared ownership. |
| **Article 9: Protection and Prevention** | Implement policies and tools to maintain confidentiality, integrity, and availability. Address technical vulnerabilities proactively. | Dynamic secrets, automated rotation, and fine-grained access policies reduce the blast radius of a leaked credential from indefinite to a bounded lease window. |
| **Article 10: Detection and Patch Management** | Track, prioritize, and evidence remediation of vulnerabilities, including unauthorized access attempts. | Centralized audit logs record every read, write, and authentication event against a secret, giving incident response teams the evidence git-crypt and similar tools cannot produce. |
| **Article 28: ICT Third-Party Risk Management** | Perform due diligence on third-party ICT providers and enforce appropriate security standards contractually. | A governed secrets manager enforces consistent access policies across every third-party integration, instead of relying on how carefully each team distributed a shared key. |

### NIS2: Directive (EU) 2022/2555

| NIS2 Article | Requirement | How a Centralized Secrets Manager Addresses It |
| :-- | :-- | :-- |
| **Article 21(2)(a): Risk Analysis and Security Policies** | Implement policies on risk analysis and information system security, proportionate to risk exposure. | A single secrets management platform is a documented, auditable control point rather than a set of ad hoc encrypted files spread across dozens of repositories. |
| **Article 21(2)(c): Incident Handling** | Maintain the ability to detect, respond to, and recover from security incidents. | Centralized access logs let a security team answer "who accessed this secret and when" within minutes, a question git-crypt style tools cannot answer at all. |
| **Article 21(2)(e): Security in Systems Acquisition and Maintenance** | Address vulnerability handling and access management across the acquisition and maintenance lifecycle of network and information systems. | Automated credential rotation and lease expiry remove long-lived static secrets from the maintenance lifecycle entirely. |
| **Article 21(2)(h): Cryptography and Encryption Policies** | Implement policies on the use of cryptography where appropriate. | A secrets manager enforces encryption at rest and in transit centrally, with key management and access policy defined once and applied everywhere, instead of per-repository GPG or age keys managed informally by whoever set them up. |

The combined scope of DORA and NIS2 covers the overwhelming majority of enterprises that operate critical or regulated services in the European Union. If your organization falls under either regulation, "we encrypt the file before committing it" is not going to satisfy an auditor asking for access logs.

---

## OpenBao: The Open Source Secrets Manager to Adopt in 2026

Once the decision to adopt a secrets manager is made, the next question is which one. In 2026, if a vendor-backed product is not mandatory for your organization, there is a clear king of the category: OpenBao.

[OpenBao](https://openbao.org/) forked the last MPL 2.0 licensed release of HashiCorp Vault after HashiCorp moved Vault to the Business Source License in 2023. It has since developed independently, maintaining broad API compatibility while opening up several capabilities that used to be Vault Enterprise-only, such as built-in namespaces. Its rise since the fork has been fast:

- **August 2023**: HashiCorp relicenses Vault (and its other core products) from MPL 2.0 to the Business Source License, closing off the last fully open source release.
- **October 2023**: the fork is announced as OpenBao, incubated under the Linux Foundation's LF Edge, with early backing from IBM, SAP, and other contributors who wanted a vendor-neutral, OSI-approved license to continue.
- **June 2024**: OpenBao's Technical Steering Committee is formally constituted, moving the project from a bootstrap phase into standing open governance.
- **July 2024**: [OpenBao v2.0.0 ships as the initial GA release](https://openbao.org/blog/release-v2-0-0/), stabilizing the fork and adding features unavailable in Vault's open edition, such as paginated list APIs.
- **May 2025**: [OpenBao moves from LF Edge to the Open Source Security Foundation (OpenSSF) as a Sandbox project](https://github.com/openbao/openbao/blob/main/GOVERNANCE.md), governed by a Technical Steering Committee with both individual and corporate members, including contributors from GitLab, SAP, and ControlPlane.
- **2025**: OpenBao is listed in the European Commission's [EU Open Source Software Catalogue](https://interoperable-europe.ec.europa.eu/eu-oss-catalogue/solutions/openbao), a signal of institutional recognition that matters directly for organizations bound by DORA or NIS2 procurement rules.

OpenBao is not a CNCF project, but it is MPL 2.0 licensed with no vendor-controlled relicensing risk, it supports dynamic secrets, leasing, and revocation, the same primitives that address the DORA and NIS2 gaps described above, and it is already running in production at organizations with real compliance obligations. For a team evaluating whether to move beyond encrypted files in Git, or looking for an open source secrets manager that will not face another licensing surprise, OpenBao is a project you can commit to today, not just a candidate to keep watching.

---

## Conclusion

git-crypt, SOPS, and sealed-secrets solve confidentiality for files at rest. They do not solve access governance, revocation, dynamic issuance, or audit logging, and the incident scenario above shows exactly where that gap turns into a real problem.

Neither objection holds up: the operational gaps are structural, not fixable by using the tools more carefully, and under DORA or NIS2 the regulatory objection was never available in the first place.

The conversation about whether to adopt a centralized secrets manager should be over in 2026. What remains is execution: inventorying where credentials currently live, including every encrypted file and every shared key, identifying which ones are effectively unmanaged, and migrating them behind a platform with a real audit trail. Start with the credentials that would cause the most damage if leaked, and measure the difference that trail makes the first time you actually need one.

---

## References

- [git-crypt](https://github.com/AGWA/git-crypt)
- [SOPS](https://github.com/getsops/sops)
- [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)
- [Digital Operational Resilience Act (Regulation EU 2022/2554)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022R2554)
- [NIS2 (Directive EU 2022/2555)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32022L2555)
- [OpenBao](https://openbao.org/)
- [OpenBao v2.0.0 GA release announcement](https://openbao.org/blog/release-v2-0-0/)
- [OpenBao governance and TSC membership](https://github.com/openbao/openbao/blob/main/GOVERNANCE.md)
- [OpenBao on the EU Open Source Software Catalogue](https://interoperable-europe.ec.europa.eu/eu-oss-catalogue/solutions/openbao)

---
