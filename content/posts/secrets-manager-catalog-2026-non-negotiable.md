---
title: "In 2026 I Am Still Asked Why You Need a Centralized Secrets Manager"
date: 2026-07-02T09:30:00+01:00
tags: [
  "secrets-management", "secrets-manager", "devsecops", "openbao",
  "hashicorp-vault", "dora", "nis2", "compliance", "kubernetes",
  "cybersecurity", "cloud-native", "git-crypt", "sops", "audit-logging", "cncf"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Why a centralized secrets manager is non-negotiable in 2026: the operational limits of git-crypt and sealed-secrets style tools, the DORA and NIS2 mandate, and why OpenBao is a serious open source candidate."
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

The honest answer is short: encryption at rest inside a Git repository solves one problem, confidentiality of the file, and leaves the harder problems untouched. European regulations like DORA and NIS2 then remove any remaining room for debate. Both arguments stand on their own. Together they leave nothing to argue about.

Let me explain each of them.

---

## The Technological Case: Encryption Is Not Governance

Tools like [git-crypt](https://github.com/AGWA/git-crypt), [SOPS](https://github.com/getsops/sops), and Kubernetes [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) solved a real and painful problem: they let teams commit encrypted secrets alongside code instead of pasting plaintext credentials into a repository, a Slack message, or a shared spreadsheet. For a small team, or for a narrow, well-scoped use case like encrypting a handful of values in a GitOps repo, they are a legitimate and pragmatic choice. I am not arguing against using them tactically.

The problem is that many organizations stop there and treat "the file is encrypted" as equivalent to "secrets are managed." It isn't, and the gap becomes visible the moment something goes wrong.

Take a scenario I have seen play out in more than one organization. Two engineers, A and B, both clone a repository protected with git-crypt. Both have a valid GPG key added to the repository's `.git-crypt` keyring, because both needed access at some point. Engineer A runs `git-crypt unlock` on a laptop to debug a deployment issue and, out of habit, leaves the repository unlocked on disk afterwards. Months later that laptop is compromised, or the repository is copied to a shared drive for a migration, or engineer A leaves the company and nobody thinks to rotate the key because nobody kept a record of who held it. When the incident response team asks "who could have accessed this secret, and did anyone actually access it," git-crypt has no answer. Decryption happens locally, offline, with no logging and no interaction with any server. There is no event to query, no timestamp, no IP address, nothing to hand to an auditor or a SOC analyst. The only trail is the Git commit history, which tells you when a file changed, never who read the decrypted content or when.

This is not a hypothetical edge case; it is the structural design of the tool. A few concrete gaps worth naming explicitly:

- **No revocation without re-encryption.** Removing a former employee's access with git-crypt means re-encrypting every protected file with a new key and redistributing it to every remaining holder. With a secrets manager, revoking one identity's access is an API call that takes effect immediately, with no impact on anyone else.
- **No dynamic secrets.** A database password stored in an encrypted file is the same password until someone manually rotates it. A secrets manager can issue a short-lived database credential on demand and revoke it automatically when the lease expires, so a leaked credential has a shelf life measured in hours instead of years.
- **No fine-grained, identity-aware access control.** git-crypt and SOPS decisions are binary: you either hold a key that decrypts the whole protected set, or you don't. There is no way to say "this service can read only its own database password" without splitting repositories or files, which quickly becomes unmanageable at scale.
- **Keys travel with people, not with policy.** Once a GPG key or an age key is distributed, the tool has no way to know where else it has been copied, whether it sits in a password manager, a CI runner's environment, or an old laptop. A secrets manager authenticates every request against an identity provider at the moment of access, not at the moment of key distribution.

None of this makes git-crypt or SOPS bad tools. It makes them exactly what they are: file encryption utilities, useful for a narrow slice of the problem. Secrets management is a different discipline, built around centralized audit logging, dynamic issuance, and revocation as a first-class operation, not an afterthought that requires a repository-wide re-key.

---

## The Regulatory Case: DORA and NIS2 Remove the Choice

If the technological argument alone is not sufficient to drive a decision for your organization, the regulatory argument is. And for a large portion of European enterprises, it is no longer a question of best practice but of legal obligation.

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

## OpenBao: A Serious Open Source Candidate

Until 2023 the default open source answer to "which secrets manager should we adopt" was almost always HashiCorp Vault. That changed when HashiCorp relicensed Vault from the Mozilla Public License to the Business Source License, a move that pushed a meaningful part of the community, and several large users, to look for a genuinely open alternative.

[OpenBao](https://openbao.org/) is that alternative. It started as a fork of the last MPL 2.0 licensed release of Vault and has since developed independently, maintaining broad API compatibility while opening up several capabilities that used to be Vault Enterprise-only, such as built-in namespaces. It is worth being precise about its governance, because this is where a lot of secondhand claims about the project get sloppy: OpenBao is not a CNCF project. It began under LF Edge and moved to the [Open Source Security Foundation (OpenSSF) as a Sandbox project in May 2025](https://github.com/openbao/openbao/blob/main/GOVERNANCE.md), where it is governed by a Technical Steering Committee with both individual and corporate members, including contributors from GitLab, SAP, and ControlPlane.

There is a data point worth citing precisely here, because it argues the license-change case better than any generic claim could. France's [Socle Interministériel de Logiciels Libres (SILL)](https://code.gouv.fr/sill/), the official catalog of open source software recommended for use across the French public administration, actually dereferenced HashiCorp Vault in August 2023, with the stated reason being the switch to the BSL. That is a concrete, government-documented example of an organization walking away from Vault specifically because of the licensing change, which is exactly the gap OpenBao was created to fill. It is a fair and accurate way to make the argument, rather than overstating OpenBao's current inclusion in that or any other official catalog, which at the time of writing it is not.

What makes OpenBao a credible option regardless of governance labels is the substance: it is MPL 2.0 licensed with no vendor-controlled relicensing risk, it supports dynamic secrets, leasing, and revocation, the same primitives that address the DORA and NIS2 gaps described above, and it is already running in production at organizations with real compliance obligations. For a team evaluating whether to move beyond encrypted files in Git, or looking for an open source option that will not face another licensing surprise, OpenBao deserves a place on the shortlist.

---

## Conclusion

If you have covered your secrets with git-crypt, SOPS, or sealed-secrets, you have solved the confidentiality problem for files at rest. You have not solved access governance, revocation, dynamic issuance, or audit logging, and the incident scenario described above shows exactly where that gap turns into a real problem.

The technical objection does not hold: the operational gaps in file-encryption tools are structural, not solvable by using them more carefully. The regulatory objection, for anyone operating under DORA or NIS2, was never available in the first place.

If you want more background on why secrets managers matter, this blog also covers the [CyberArk Conjur enterprise angle](/posts/why-you-need-kubernetes-secrets-manager), a [recent zero-day incident across two major secrets manager vendors](/posts/0day-cves-secrets-manager), and the ongoing [maintenance challenges facing the External Secrets Operator project](/posts/external-secrets-operator-team-needs-help), which is often the missing piece connecting a secrets manager to Kubernetes workloads.

The conversation about whether to adopt a centralized secrets manager should be over in 2026. What remains is execution: inventorying where credentials currently live, including every encrypted file and every shared key, identifying which ones are effectively unmanaged, and migrating them behind a platform that can answer who accessed what, and when. Start with the credentials that would cause the most damage if leaked, and measure the difference an audit log makes the first time you actually need one.

---
