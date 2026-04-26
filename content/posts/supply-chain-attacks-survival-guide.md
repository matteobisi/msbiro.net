---
title: "Supply Chain Attacks Keep Working: A Practical Survival Guide"
date: 2026-04-26T12:00:00+01:00
tags: [
  "cybersecurity", "supply-chain-security", "devsecops",
  "open-source", "devops", "github-actions", "npm"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Bitwarden CLI, Trivy, Axios, Notepad++: supply chain attacks are becoming routine. This is not another incident report. It is a set of practical ground rules to minimize the blast radius when your luck runs out."
canonicalURL: "https://www.msbiro.net/posts/supply-chain-attacks-survival-guide/"
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
    alt: "Supply Chain Attacks Keep Working: A Practical Survival Guide"
    caption: "Because the question is no longer if, but when"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## It Happened Again

On April 22, 2026, the official Bitwarden CLI npm package (`@bitwarden/cli`) was compromised. For roughly 90 minutes, between 5:57 PM and 7:30 PM ET, anyone who ran `npm install @bitwarden/cli` received a malicious package. Around 334 developers did exactly that.

The attackers did not break into Bitwarden's npm account directly. Instead, they hijacked a GitHub Actions workflow in Bitwarden's CI/CD pipeline and weaponised npm's Trusted Publishing mechanism to push a poisoned release. Trusted Publishing is OIDC-based and requires no stored credentials: it was introduced as a hardening measure after credential-based attacks. It became the entry point.

The malicious preinstall scripts (`bw1.js` and `bwsetup.js`) ran before anyone typed a single CLI command. They harvested SSH keys, GitHub tokens, npm tokens, cloud provider credentials, `.env` files, and more: anything sitting in the environment. They also specifically targeted credentials for AI coding tools: Claude, OpenAI Codex, Gemini CLI, Cursor, and others. The exfiltrated data was encrypted and sent to `audit.checkmarx[.]cx`, a typosquatted domain, and written to Dune-themed public repositories created in the victims' own GitHub accounts. If a valid GitHub token was found, the malware attempted to inject malicious workflows into other repositories, making the blast radius potentially worm-like within an organisation.

The attack is attributed to a group called "TeamPCP" and is part of a broader campaign named "Shai-Hulud" (a Dune reference), which has been running for months.

Bitwarden detected and yanked the package. A clean version followed. The incident window was 90 minutes.

But I am not here to write another incident report. What I want to do is step back, look at the pattern, and talk about what you can actually do about it.

---

## This Is Not New: The Pattern of the Last Few Months

The Bitwarden attack is the latest link in a chain. The months leading up to it were dense with similar incidents.

**Notepad++** was targeted by the Lotus Blossom group (also known as Lotus Panda) between mid-2025 and early 2026. The attackers did not exploit a bug in the editor itself: they compromised the shared hosting infrastructure and hijacked the update channel. A small number of targets, mainly government, telecom, and critical infrastructure organisations in Southeast Asia and Central America, received trojanised installers carrying Cobalt Strike beacons and custom backdoors. The general public was not affected, but the technique was precise and persistent: the attackers maintained access for six months using stolen internal credentials.

**Trivy**, Aqua Security's widely used container vulnerability scanner, was compromised in March 2026. The GitHub Actions pipeline was hijacked via spoofed release tags, and poisoned Docker images were distributed. Because Trivy is used inside build pipelines at thousands of organisations, the attack had an outsized potential blast radius: a compromised scanner sitting inside your CI/CD is not just a threat to the build; it is a threat to everything the build touches. Aqua and the community caught it. The campaign is attributed to the same TeamPCP group.

**Axios**, the JavaScript HTTP library with well over 100 million weekly downloads, was hit in the same March 2026 wave. A maintainer account on npm was compromised, and a malicious release was pushed with a postinstall credential stealer targeting AWS and GCP secrets, Kubernetes credentials, and any cloud service token present in the environment. Given Axios's download numbers, even a brief window of exposure has enormous reach. LiteLLM and Telnyx fell in the same wave, with attackers cascading from one compromised project to the next using credentials stolen along the way.

The pattern across all of these runs the same way: trusted update channels, legitimate-looking releases, interpreted payloads (JavaScript or Python), and credential theft that targets developer environments and CI/CD pipelines. The goal is rarely to destroy anything immediately; it is to get persistent access to the secrets that open the next door.

---

## EDR Is Not Enough at the Package Layer — But It Is Not Irrelevant Either

This is the uncomfortable part. Most organisations run endpoint detection and response platforms: CrowdStrike, SentinelOne, Microsoft Defender, and similar tools. They are well-funded, well-marketed, and genuinely effective at what they were built to do.

The problem is that they were not built for the package installation layer.

I came across an interview on [Laterstack](https://laterstack.com/edr-open-source-malware-paul-mccarty-interview/) with Paul McCarty, a supply chain security researcher who was part of the group that caught the Trivy compromise before the major vendors published anything. His explanation of why EDR misses interpreted malware is worth reading in full, but the short version is this:

EDR platforms are built around two detection primitives. The first is file hashes of known bad binaries: if they have seen a bad `.exe` or `.dll` before, they will catch it again. The second is anomalous behaviour signatures like mass file encryption. Both are irrelevant when the attacker ships a malicious `.js` or `.py` postinstall script. There is no binary to hash: a threat actor can push twenty new versions of a malicious JavaScript payload in a single day, and each one will have a different hash. The behaviour of an infostealer does not look like ransomware: it reads a few files in well-known browser or credential directories, serialises them into a JSON blob, and POSTs them over HTTPS. That is exactly what legitimate apps do constantly.

McCarty ran thousands of interpreted payloads through VirusTotal and the major sandboxes. The EDR vendors did not detect them. The only exception he found was Kaspersky, and even that was rudimentary.

If you are running `npm install` or `pip install` in a developer environment or a CI/CD pipeline, your EDR vendor is not meaningfully protecting that operation. The assumption that it is protected is the assumption that gets organisations into headlines.

The Axios compromise shows where EDR can still contribute. The attack did not stop at a JavaScript package: it chained into a cross-platform binary RAT (WAVESHAPER.V2), deployed via a renamed PowerShell executable on Windows. That binary stage is exactly what behavioural EDR was built for. SentinelOne's Lunar engine caught the renamed binary execution technique regardless of payload hash, and their Wayfinder team ran proactive hunts across all MDR regions using specific IOCs within hours of the attack being confirmed.

The lesson is architectural: EDR provides no meaningful protection at the interpreted-package stage, but it can intercept the binary payload stage that follows — if the attack chain includes one, and if the EDR is configured to act autonomously at machine speed rather than waiting for a human analyst. The gap is still real. It is just not the entire picture.

---

## Ground Rules for When Your Luck Runs Out

I want to be direct: there is no configuration that makes you immune. The Bitwarden attack abused a legitimate publishing mechanism. The Axios attack used a compromised maintainer account. The Trivy attack spoofed a release tag. Each of these vectors looks legitimate at the moment it fires.

What you can do is reduce the blast radius and speed up detection. Here is what I consider the practical baseline.

### 1. Pin GitHub Actions to a Commit SHA

Tags in GitHub Actions are mutable. When you write `uses: actions/checkout@v4`, you are trusting whoever controls that tag to have not moved it to a different commit. The Bitwarden attack exploited this exact primitive: a hijacked workflow step, referenced by tag, became the entry point.

The correct form is:

```yaml
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

The comment preserves readability; the hash enforces integrity. This is not theoretical hygiene: it is the specific control that would have raised a red flag in the Bitwarden case. Tools like [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot) and [StepSecurity Harden-Runner](https://github.com/step-security/harden-runner) can automate the pinning and keep the hashes current.

### 2. Enforce Least-Privilege Token Permissions on Every Workflow

The Bitwarden attacker used the `GITHUB_TOKEN` in the hijacked workflow to publish to npm. Default GitHub Actions token permissions are broader than most workflows need. Set explicit permissions at the workflow level and override at the job level only where necessary:

```yaml
permissions: {}

jobs:
  publish:
    permissions:
      contents: read
      id-token: write # only if using OIDC trusted publishing
```

Workflow permissions that are not explicitly granted cannot be abused.

### 3. Use MFA on Every Registry Account

npm, PyPI, Docker Hub, and every other package registry account should have multi-factor authentication enabled and enforced. The Axios attack started with a compromised maintainer account; MFA would not have made that impossible, but it raises the cost significantly. npm has supported hardware key and TOTP MFA for years, and for accounts with publish access it can be made mandatory at the organisation level.

This is table-stakes hygiene that still gets skipped.

### 4. Treat Postinstall Scripts as Hostile Until Proven Otherwise

The Bitwarden malware ran before a single CLI command was issued, through npm's `preinstall` lifecycle hook. Every package that ships install-time scripts deserves a question: does this package actually need to run code during installation?

For dependencies you own or control, remove postinstall scripts unless they are strictly necessary. For third-party dependencies, `npm install --ignore-scripts` prevents lifecycle hooks from executing. This is not always practical for complex toolchains, but in CI/CD environments where you are installing a known, locked dependency set, it is a reasonable default to consider.

### 5. Lock and Verify Your Dependency Tree

`package-lock.json`, `yarn.lock`, `poetry.lock`, and their equivalents exist for a reason. A locked dependency tree means that an attacker who publishes a malicious patch version will not automatically land in your pipeline. Commit your lockfiles, configure your CI to fail if the lockfile is out of sync with the manifest, and review lockfile diffs in pull requests: they are often where supply chain injections hide.

For higher-stakes environments, go further. Tools like [Socket](https://socket.dev/) and [Endor Labs](https://www.endorlabs.com/) analyse packages at install time and flag newly introduced install scripts, unusual network activity patterns, and known malicious behaviour. They are not EDR; they are purpose-built for the interpreted-package threat model.

### 6. Know Your Secrets Before You Need to Rotate Them

When the Bitwarden window closed, Bitwarden published a list of what should be considered compromised: SSH keys, GitHub tokens, npm tokens, cloud credentials, anything in the environment during install. The advice was to rotate everything immediately.

That is correct advice, but it assumes you know what you have and where it lives. If you cannot enumerate all the secrets in a developer environment in under an hour, rotation under pressure becomes a guessing game. The right time to build that inventory and those rotation runbooks is before the incident.

Use a secrets manager (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) with short-lived credentials wherever possible. A token that expires in an hour is far less valuable to an attacker than one with no expiry. Enable audit logging on every secrets backend so you can tell whether a stolen credential was actually used.

### 7. Signal-Share and Monitor the Community

Paul McCarty's group caught the Trivy compromise before CrowdStrike, Wiz, and Socket had published anything. How? Not through better tools: through an intel-sharing network of researchers who communicate directly and quickly. The big vendor platforms caught up later.

This has a practical implication. Follow the communities that do this work: [OpenSourceMalware.com](https://opensourcemalware.com), the [Open Source Security Foundation](https://openssf.org), Endor Labs' blog, and the security advisories for every critical dependency in your tree. Subscribe to the GitHub Security Advisories feed for your language ecosystem. Set up Dependabot alerts and actually triage them. The signal exists; the question is whether you have a pipeline to receive it.

---

## A Note on Trust Models

McCarty made one other point in that interview that I keep thinking about. Git was not built with security in mind. It accepts user identity implicitly. You can push a commit to GitHub attributed to Linus Torvalds. The commit history that GitHub surfaces on every contributor's profile is something you can fabricate entirely on your local machine and push without challenge.

There is a project trying to fix this at the infrastructure level rather than the convention level. [gittuf](https://gittuf.dev/) is a platform-agnostic security layer for Git, incubating under the OpenSSF Supply Chain Integrity Working Group. The core problem it addresses: your repository's security policies, branch protections, and reviewer requirements all live on the forge (GitHub, GitLab, wherever you host). That makes the forge a single point of trust. If the forge is compromised, or if you migrate platforms, those controls either evaporate or require you to blindly trust whatever the new forge says.

gittuf moves security policy into the repository itself, signed and verifiable by anyone with access to the repo, independent of any hosting platform. It builds on The Update Framework (TUF) for key management and rotation, handles granular write access rules per branch, tag, or file path, and tracks the full history of policy changes inside the repository.

At [SecurityCon 2026](https://www.youtube.com/watch?v=EwZhvbmaRkk&t=1263s), the project team presented how gittuf addresses exactly the weakness McCarty described: forges as an implicit, unverifiable trust anchor. The project is currently in beta, but it is worth watching closely. The Bitwarden attack succeeded in part because the forge's Trusted Publishing mechanism was the trust anchor, and that anchor was subverted. Decentralising that trust into the repository itself is not a patch; it is a structural rethink.

The trust we place in signed releases, pinned hashes, and verified publishers is not perfect: the Bitwarden attack abused a legitimate publishing mechanism. But each layer of verification raises the cost and complexity of a successful attack. The goal is not zero risk; it is making the operation expensive enough that attackers move toward softer targets.

The software supply chain has been a primary attack surface for years. The incidents of the last few months just make the pattern impossible to keep ignoring. The tools that catch these attacks are not the tools most organisations have deployed. The people who catch these attacks are underfunded and operating at the margins.

That is the threat model. Build your defences accordingly.

---

## References

- [Endor Labs: Shai-Hulud, the Bitwarden CLI Supply Chain Attack](https://www.endorlabs.com/learn/shai-hulud-the-third-coming----inside-the-bitwarden-cli-2026-4-0-supply-chain-attack)
- [OX Security: Bitwarden CLI, Inside the Shai-Hulud Attack](https://www.ox.security/blog/shai-hulud-bitwarden-cli-supply-chain-attack/)
- [StepSecurity: Bitwarden CLI Hijacked on npm](https://stepsecurity.io/blog/bitwarden-cli-hijacked-npm/)
- [Laterstack: EDR, Open Source Malware, and the People Gap (Paul McCarty interview)](https://laterstack.com/edr-open-source-malware-paul-mccarty-interview/)
- [Cloud Security Newsletter: Supply Chain Attack on Trivy, LiteLLM and Axios](https://www.cloudsecuritynewsletter.com/p/supply-chain-attack-trivy-litellm-axios-appsec-ai-2026)
- [SentinelOne: How SentinelOne's AI EDR Stops the Axios Attack Autonomously](https://www.sentinelone.com/blog/securing-the-supply-chain-how-sentinelones-ai-edr-stops-the-axios-attack-autonomously/)
- [The Hacker News: Bitwarden CLI Compromised in Ongoing Supply Chain Attack](https://thehackernews.com/2026/04/bitwarden-cli-compromised-in-ongoing.html)
- [gittuf: Platform-Agnostic Git Security (OpenSSF)](https://gittuf.dev/)
- [SecurityCon 2026: gittuf, Removing the Forge as a Single Point of Trust](https://www.youtube.com/watch?v=EwZhvbmaRkk&t=1263s)
