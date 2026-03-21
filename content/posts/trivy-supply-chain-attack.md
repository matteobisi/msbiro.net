---
title: "The Trivy Supply Chain Attack: A Breakdown of Credential Theft and the CanisterWorm Escalation"
date: 2026-03-19T10:32:37Z
tags: ["trivy", "supply-chain", "github-actions", "cloud-native", "kubernetes", "cybersecurity", "open-source", "devops", "containers"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A comprehensive analysis of the March 2026 Trivy supply chain incident: from malicious GitHub Actions to the self-propagating CanisterWorm."
canonicalURL: "https://www.msbiro.net/posts/trivy-supply-chain-attack/"
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
    alt: "Trivy supply chain incident illustration"
    caption: "The March 2026 supply chain compromise targeted one of the most trusted security tools in the cloud-native ecosystem."
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

Trivy, the widely adopted open-source security scanner from Aqua Security, is a cornerstone of modern CI/CD pipelines and container security. With over 33,000 stars on GitHub as of March 2026, its footprint spans across Docker images, Homebrew, and countless developer machines. This ubiquity, however, made the supply-chain compromise discovered between March 19–21, 2026, particularly devastating.

The incident was not a single point of failure but a multi-stage attack involving malicious releases, manipulated GitHub Actions, and a self-propagating worm that leveraged decentralized infrastructure.

## Phase 1: The Initial Compromise

The attack began when threat actors gained access to official Trivy distribution channels. This allowed them to execute a two-pronged attack on both binaries and automation workflows.

### 1. The Malicious Release (v0.69.4)
Attackers published a malicious Trivy release, version **v0.69.4**. While appearing legitimate, the binary contained a hidden payload designed for credential harvesting.

### 2. GitHub Actions Tag Hijacking
Simultaneously, the attackers force-pushed many tags in the `aquasecurity/trivy-action` and `aquasecurity/setup-trivy` repositories. By redirecting mutable version tags (like `@v0.18`) to malicious commits, they ensured that any pipeline pulling these actions would automatically ingest the payload without developer intervention.

## The Payload: Harvesting and Persistence

Once executed within a GitHub Actions runner or a developer machine, the malicious code initiated a series of exfiltration and persistence maneuvers.

*   **Credential Harvesting:** The malware targeted runner memory, environment variables, and local sensitive files. It specifically looked for SSH keys, cloud provider credentials (AWS/GCP/Azure), Kubernetes tokens, and crypto wallets.
*   **Stealthy Exfiltration:** Stolen data was encrypted and sent to a typosquatted domain: `scan.aquasecurtiy[.]org` (note the intentional misspelling). If this failed, the malware attempted to create a repository named `tpcp-docs` within the victim's GitHub account to act as a data drop.
*   **Persistence via ICP Canisters:** On developer machines, the malware wrote a Python dropper and a `systemd` user service. This service polled an **Internet Computer (ICP)** canister—a decentralized "dead-drop"—to fetch next-stage payloads. Using a decentralized canister makes takedown efforts significantly harder for security teams.

## Phase 2: Escalation to 'CanisterWorm'

The attack reached a critical turning point when the stolen credentials were used to compromise the npm ecosystem. This birthed what researchers have dubbed the **CanisterWorm**.

1.  **npm Propagation:** Attackers used stolen npm tokens to push malicious versions of several popular packages.
2.  **The Worm Mechanism:** These packages included `postinstall` hooks that dropped a Python backdoor. This backdoor didn't just exfiltrate data; it actively searched the infected host for *additional* npm tokens.
3.  **Self-Propagation:** If a token was found, the worm would programmatically publish itself as a new version of any package the token had permissions for. This transformed a localized breach into a self-propagating supply-chain infection.

## Why This Matters: A New Blueprint for Attacks

The Trivy incident is a wake-up call for several reasons:
*   **Abuse of Trust in Mutable Tags:** The reliance on version tags instead of commit SHAs remains a massive blind spot in CI/CD security.
*   **Decentralized C2 Infrastructure:** The use of ICP canisters demonstrates how attackers are moving away from traditional VPS-based Command & Control (C2) to more resilient, decentralized alternatives.
*   **Automated Propagation:** The CanisterWorm shows how quickly a "data theft" incident can escalate into a global supply chain crisis when automation is turned against itself.

## Incident Response & Hunting Guide

If you used Trivy v0.69.4 or the affected Actions between March 19–21, 2026, **assume compromise.**

### Immediate Mitigation Checklist
- [ ] **Rotate All Secrets:** Immediately rotate any credentials (Cloud, K8s, SSH, npm, GitHub) that were accessible to your CI/CD pipelines or developer environments during the window.
- [ ] **Purge Artifacts:** Clear build caches and container registries of any Trivy v0.69.4 artifacts.
- [ ] **Update and Pin:** Move to the vendor-verified safe versions and **pin all GitHub Actions to full commit SHAs** rather than tags.
- [ ] **Audit GitHub Orgs:** Search for unexpected repositories named `tpcp-docs`.

### Indicators of Compromise (IoCs)
*   **Network Calls:** `scan.aquasecurtiy[.]org` or the IP `45.148.10[.]212`.
*   **ICP Traffic:** Traffic to the canister URL: `tdtqy-oyaaa-aaaae-af2dq-cai.raw.icp0.io`.
*   **Persistence:** Unusual `systemd` user services or unexplained Python processes polling external URLs.

## Conclusion

The Trivy supply chain attack highlights the fragility of our collective trust in common developer tooling. While immediate containment is critical, the long-term solution lies in **Zero Trust for Build Pipelines**: pinning dependencies to immutable hashes, enforcing least-privilege for automation tokens, and monitoring for anomalous outbound traffic from CI runners.

### References
- [The Hacker News — Trivy GitHub Actions compromise](https://thehackernews.com/2026/03/trivy-security-scanner-github-actions.html)
- [The Hacker News — CanisterWorm and npm propagation](https://thehackernews.com/2026/03/trivy-supply-chain-attack-triggers-self.html)
- [Wiz blog analysis — Trivy Compromised: TeamPCP Attack](https://www.wiz.io/blog/trivy-compromised-teampcp-supply-chain-attack)

---

*This post is a breakdown of the March 2026 incident based on available security research.*
