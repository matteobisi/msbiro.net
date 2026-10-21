---
title: "OVHcloud CLI Supply Chain Compromise: A User Playbook"
date: 2026-10-21T07:00:00+02:00
tags: [
  "supply-chain", "supply-chain-security", "cybersecurity", "devsecops",
  "open-source", "github", "incident-response", "ovhcloud", "malware", "cli"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "On 19 October 2026, OVHcloud CLI GitHub releases were replaced with a credential stealer. Check if you were exposed, and a playbook for the next poisoned CLI."
keywords: [
  "OVHcloud CLI supply chain", "ovhcloud-cli compromise", "GitHub release malware",
  "CLI credential stealer", "GlassWorm OVHcloud", "how to check ovhcloud-cli",
  "supply chain incident response", "poisoned CLI playbook", "developer laptop secrets",
  "pin SHA-256 CLI install", "verify release binary provenance", "reproducible builds"
]
canonicalURL: "https://www.msbiro.net/posts/ovhcloud-cli-supply-chain-compromise-user-playbook/"
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
    alt: "OVHcloud CLI GitHub release binaries replaced with a credential stealer in October 2026"
    caption: "The first hour is the part that repeats."
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

On 19 October 2026, a supply chain compromise replaced GitHub release binaries of the official OVHcloud CLI (`ovhcloud-cli`) with a credential stealer for about ninety minutes. Anyone who installed from GitHub, Homebrew, or the project's install script in that window could have received malware instead of a cloud tool. The source repository was clean. The build pipeline was clean. The files on the release page were not.

[OpenSourceMalware reconstructed the incident](https://opensourcemalware.com/blog/glassworm-hacks-ovh-clouds-cli-tool): timeline, how the assets were swapped, how the malicious build was identified, GlassWorm indicators, hashes. OVHcloud published a [security advisory](https://github.com/ovh/ovhcloud-cli/security/advisories/GHSA-862f-55f5-vmvw) the next day, pulled the bad files the same afternoon, and shipped a clean immutable v0.16.0.

*Updated 21 October 2026: added a section on clean source vs. poisoned release binaries, and why a pinned hash only helps if you verified what you pinned. Thanks to a reader on LinkedIn for pointing out the gap.*

The rest of 2026 looks like this too. Bitwarden, Trivy, Axios, LiteLLM, Notepad++, now a cloud CLI. A trusted install path, a short window, a payload that reads the laptop, then a scramble to rotate secrets nobody had listed. What I want on the desk now is a first-hour sequence, because the next name will be different.

---

## How to check if you installed the malicious OVHcloud CLI

You were exposed only if you installed or upgraded `ovhcloud-cli` between 12:47 and 14:21 UTC on 19 October 2026, versions v0.11.0 through v0.15.0. If you did not touch the tool that day, skip this incident.

Do not run the binary to check. Use the install timestamp, look in the temp directory for a folder named `envchecker-` followed by digits, and compare the file on disk with the hashes in the [advisory](https://github.com/ovh/ovhcloud-cli/security/advisories/GHSA-862f-55f5-vmvw). Indicators and forensics are in the [OpenSourceMalware write-up](https://opensourcemalware.com/blog/glassworm-hacks-ovh-clouds-cli-tool). If any of that is ugly, treat the machine as compromised and start below. Confirmation can wait.

---

## What to do after a CLI supply chain compromise

"Rotate everything" is correct and also useless at 16:00 on a Tuesday. It does not say what comes first or which machine you should be sitting at. This is the order I want in my head the next time a CLI, plugin, library, or image I installed is reported as poisoned.

### Isolate the machine right now

Stop using the machine for anything that touches a password or a token. Disconnect it from the network. Do not wipe it yet. The artifact, its timestamp, the package manager log, and your shell history are what tell you how much was exposed.

Write down three answers before you rotate anything: which tool and version landed, at what time, and which credentials your user could read in that session. Environment variables, cloud CLI configs, kubeconfigs, SSH keys, editor stores, coding-agent state. If you cannot list them, you are already guessing.

### Rotate credentials from a clean device

Revoke in this order, because a stolen session can undo a reset you just performed:

1. Identity provider sessions and OAuth grants. Everything else hangs off these.
2. Code hosting sessions, personal access tokens, and SSH keys. Repositories and releases can be rewritten with them.
3. The credentials of the service the compromised tool manages directly.
4. Other cloud providers, Kubernetes configs, and service accounts reachable from the same laptop.
5. Package registry tokens, which would turn this into someone else's supply chain problem.
6. CI/CD secrets and everything that lives in a secrets manager.

Revoke sessions, not just passwords. A password reset that leaves an active refresh token evicts nobody. While you are in each system, copy the activity log for the time between install and revocation before it rolls over.

### Rebuild the laptop within a day

Rebuild the machine from scratch. A laptop that ran an unknown payload under your user account is not a machine you clean. Restore data from a backup you trust, and re-issue credentials rather than bringing the old ones back. Install the replacement against a digest you verified independently of the page that served the first file (see the next sections).

### Write the timeline within a week

Write the timeline while you still remember it, including the parts you could not determine. Decide whether the incident is notifiable, and tell the teams whose systems those credentials could reach. Change one thing in your own setup that would have made this faster.

---

## Clean source, poisoned binary: verify before you pin

Clean source, like the OVH cli case, does not mean a clean binary. In this case the repository and the tags were fine; the attacker replaced the files on the release page, including `checksums.txt`, so the published hashes matched the malware. Reading the code or checking the hash next to the download told you nothing.

That also limits what pinning can do. A pinned SHA-256 guarantees you keep getting the same bytes, not that those bytes were good. If you record the hash of a file you downloaded inside the window, you pin the malware. Pin only what you verified, against something the attacker could not also write to:

1. **Rebuild and compare.** Build the tagged source and compare digests. Works only when the project builds reproducibly, but it is the only check that ties the binary to source you can read.
2. **Verify build provenance.** An attestation (SLSA, GitHub artifact attestations, Sigstore) is signed by the CI workflow and records the digest, the commit, and the workflow that built it. A file uploaded by hand cannot carry a valid one. Check the expected repository and workflow, not just "is it signed", e.g. `gh attestation verify <file> --repo owner/tool`. It does not help if the pipeline itself is compromised, and it only exists if the project publishes it. If a tool that holds your keys does not, ask for it.
3. **Use an independent channel.** A package manager that builds from source or checks source hashes on its own (for Go tools, `go install` with the Go checksum database), or a digest you recorded before the incident.
4. **`checksums.txt` on the same page.** Catches a broken download, not an attacker with write access to the release.

Then pin: in CI, check downloaded CLIs against a SHA-256 stored in your repository and fail on mismatch; pin images by `@sha256:` and GitHub Actions by commit SHA; make every upgrade a reviewed change of digest, verified as above. Pinning catches a file that changes after you trusted it, which is how this one was caught: a CI job with a hash recorded before the swap failed. Provenance catches a file the project never built. You want both.

---

## Reduce exposure before the next poisoned CLI

Upgrade tools that hold keys as a decision, not as background noise. Keep less to steal: short-lived credentials through single sign-on, scoped keys with an expiry, no long-lived secrets in dotfiles on the laptop you also browse from. If a single binary can read your entire cloud access, the release page is not your weakest link.

Watch the release pages of the tools that hold keys the way you already watch CVEs. The payload in this incident went after VS Code, Cursor, Claude, OpenCode, cloud CLIs, SSH, Terraform, and kubeconfigs. That list is the machine we actually use. I covered pipelines and registries in [Supply Chain Attacks Won't Stop](/posts/supply-chain-attack-prevention-8-controls/), and the workstation in [Shift Left Starts in the IDE](/posts/shift-left-starts-in-the-ide/).

---

## Takeaways

You will not catch every supply chain hit in time. Verify what you install against the build, not the release page, then pin it. Keep fewer long-lived secrets within reach of one program, and have a rotation order plus a clean device ready before you need them.

The difference between a bad afternoon and a company-wide incident is rarely the sophistication of the malware. It is how much a single program could read, and how quickly you could revoke it.

---

## References

- [OpenSourceMalware: Glassworm Hacks OVHcloud's CLI Tool](https://opensourcemalware.com/blog/glassworm-hacks-ovh-clouds-cli-tool) (full incident analysis)
- [OVHcloud security advisory GHSA-862f-55f5-vmvw](https://github.com/ovh/ovhcloud-cli/security/advisories/GHSA-862f-55f5-vmvw)
- [ovh/ovhcloud-cli issue #300](https://github.com/ovh/ovhcloud-cli/issues/300)
- [SLSA: Supply-chain Levels for Software Artifacts](https://slsa.dev/)
- [Reproducible Builds](https://reproducible-builds.org/)
- [GitHub CLI: gh attestation verify](https://cli.github.com/manual/gh_attestation_verify)
- [msbiro.net: Supply Chain Attacks Won't Stop, 8 Controls to Reduce Your Exposure](https://www.msbiro.net/posts/supply-chain-attack-prevention-8-controls/)
- [msbiro.net: Shift Left Starts in the IDE](https://www.msbiro.net/posts/shift-left-starts-in-the-ide/)
