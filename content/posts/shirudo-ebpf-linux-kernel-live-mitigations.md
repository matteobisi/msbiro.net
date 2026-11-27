---
title: "Shirudo: eBPF Shields for the Linux Kernel in the Age of AI-Driven Vulnerabilities"
date: 2026-11-27T06:00:00Z
tags: ["linux", "container-security", "cybersecurity", "devsecops", "ai-security", "vulnerability-management"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Cisco and OpenAI recently announced Shirudo, an eBPF-based security platform for Linux. This article looks at how signed upatches, TPM attestation, and pull-based delivery can help defenders react faster when AI compresses the vulnerability discovery timeline."
canonicalURL: "https://www.msbiro.net/posts/shirudo-ebpf-linux-kernel-live-mitigations/"
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
    alt: "Shirudo eBPF shields for Linux kernel live mitigations"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

A few days ago, in [Four Critical Flaws in Mature Software](/posts/ai-changing-vulnerability-discovery-four-critical-flaws/), I looked at a troubling pattern: AI-assisted research is making vulnerability discovery faster, cheaper, and more repeatable. The four cases I covered were not exotic bugs in obscure software. They lived in mature, widely deployed code, and in some cases multiple teams found the same issue almost at the same time. My conclusion was simple: the bottleneck is moving from discovery to remediation.

That is why the recent announcement of [Shirudo](https://github.com/shirudo-dev/shirudo) by Cisco and OpenAI caught my attention. Shirudo is an eBPF-based security platform for Linux. It is not a replacement for patching, and it is not a magic shield that makes vulnerable systems safe forever. What makes it interesting is the direction: it tries to give defenders a way to apply signed, verifiable runtime mitigations quickly, while the normal patch and reboot cycle is still running.

I have not put hands on Shirudo in a real environment yet. I spent time with the [Isovalent getting started lab](https://isovalent.com/labs/shirudo-getting-started/), which is a good way to understand the workflow without building a full test environment from scratch. With my team, we plan to explore and deep dive into this technology later this year. This article is therefore an early, informational look at what Shirudo is, why it matters, and which questions CIOs and CTOs should ask before getting excited.

## Why Shirudo matters now

The problem I described in the previous article is mostly a timing problem. A vulnerability becomes public, or becomes practically exploitable, before most organizations can patch every affected system. The fix may already exist upstream, but production reality is slower: maintenance windows, reboot cycles, dependency checks, change management, legacy workloads, and third-party appliances all add friction.

AI changes the pressure on that window. When discovery becomes cheaper, defenders should expect more findings in mature components, more parallel discovery, and less time between disclosure and weaponization. In that world, "patch everything immediately" is still the right goal, but it is rarely an operational plan.

Shirudo is aimed directly at this gap. The project describes upatches as small, logically contained programs that can be delivered to running machines without a reboot. In plain terms, it gives operators a way to block or observe specific dangerous paths at runtime, using eBPF, while waiting for the proper upstream fix to be deployed.

The intuitive analogy is a kind of WAF for the Linux kernel, but the analogy is only useful if we stay precise. Shirudo is not filtering HTTP requests at an edge proxy. It operates at the Linux and workload runtime level, with signed BPF loaders, kernel enforcement, and TPM-backed evidence.

## What Shirudo actually is

At a high level, Shirudo has two roles: an operator side and a target side. The operator builds, signs, and publishes upatches. The target receives, verifies, loads, and measures them. The [official documentation](https://docs.shirudo.dev/) describes Shirudo as a BPF security platform that restricts BPF program loads from userspace to signed loaders after boot-time initialization. That restriction is a central point. BPF is powerful, but power without control becomes risk.

The main artifact is the upatch. A BPF upatch is a signed program that can attach to kernel hooks or function boundaries, depending on the use case. Shirudo also supports Java agent upatches, which are verified in userspace and loaded by configured JVMs at startup. The platform packages these artifacts, distributes them through OCI registries, and keeps an auditable trail of what was loaded.

The project highlights a few key properties:

| Area | What Shirudo brings | Why it matters for leaders |
| --- | --- | --- |
| Signed delivery | Upatches and loaders are signed; the kernel verifies BPF loads against the sealed keyring | Reduces the risk of unauthorized runtime code injection |
| Live mitigation | BPF upatches can be loaded without rebooting the machine | Shortens the time between knowing a dangerous path and containing it |
| TPM measurement | Load, unload, tree operations, and key changes extend a TPM register | Gives verifiable evidence of what was applied on a target |
| Pull-based delivery | Targets pull packages from OCI registries; no daemon, no listening socket, no push channel | Fits controlled infrastructure workflows, but requires operational design |
| Observability export | Map contents and status can be exported to OpenTelemetry collectors | Makes runtime mitigations visible to platform and security teams |

## The workflow, from fix to measured load

The most useful part of the Isovalent lab is that it makes the Shirudo workflow concrete. It is not "upload script, run script, hope for the best". The flow is deliberately structured around build, signing, publication, verification, and attestation.

The simplified lifecycle looks like this:

1. **Build the upatch.** An engineer or security team writes a small upatch and describes it through Shirudo build metadata.
2. **Sign it.** The operator signs the artifact. The signature proves who built the code and that it was not tampered with.
3. **Publish it.** The package is pushed to an OCI registry, where targets can pull it.
4. **Verify and load.** On the target, Shirudo verifies the package before loading the BPF upatch. The kernel accepts BPF only through the signed loader path.
5. **Measure it.** The load is recorded in a TPM-backed measurement chain.
6. **Attest it.** An operator can later verify TPM-signed evidence offline against the artifacts it shipped.

This workflow matters because speed without trust is dangerous. In incident response, teams often accept risky temporary controls because they have no better option. Shirudo's design tries to make the temporary control auditable: who signed it, what was loaded, when it changed, and whether the target can prove its state.

The TPM angle is especially relevant for regulated or high-assurance environments. It moves the conversation from "we think the mitigation is there" to "the target can produce evidence that this mitigation was measured". That does not solve every governance problem, but it is a better starting point than ad hoc shell scripts.

## Where it can help, and where it cannot

The use cases described by the project are broad: runtime security and observability, live mitigations, system hardening, and embedded or edge devices. For a CIO or CTO, I would translate them into three practical scenarios.

First, **emergency containment**. A critical Linux or runtime vulnerability is disclosed, exploitation is plausible, and the patch window is longer than the business can tolerate. A carefully written upatch could block the vulnerable path or narrow the attack surface until the patched kernel or package is rolled out.

Second, **targeted hardening**. Some environments need specific runtime controls that are hard to express with generic configuration. BPF-based controls can enforce narrow policies close to the kernel, with less overhead than many user-space workarounds.

Third, **fleet evidence**. For organizations that struggle to prove runtime state during audits or incident reviews, TPM-backed attestation can become a useful control. The value is not only technical; it is operational accountability.

There are also clear limits, and they should be stated plainly.

Shirudo does **not** replace kernel updates. A live mitigation buys time; it does not remove the need to deploy the fixed software. The signature proves provenance, not correctness. A signed but bad upatch can still be wrong. The kernel verifier checks memory safety properties, but it does not validate whether the mitigation is logically correct or complete.

Shirudo also does not defend against a compromised kernel, BPF verifier, TPM, or hardware. The [threat model](https://docs.shirudo.dev/security/threat-model.html) makes this boundary explicit, and that honesty is a good sign.

## The practical catch: system requirements

This is the part I would put in front of every infrastructure leader early in the conversation. Shirudo is promising, but it is not something you casually install on every existing Linux server next Monday.

The documentation is clear: production targets require a Linux kernel with the signed-loader ABI and Shirudo kernel patches, plus TPM 2.0. The project is pre-release. It runs on a bpf-next development kernel with specific capabilities and patches; a release number alone is not enough to know whether a kernel is compatible. In other words, current mainstream production fleets will likely not be ready out of the box.

That does not make Shirudo irrelevant. It makes adoption a roadmap decision. The right question is not "can we deploy this everywhere now?" The better question is "where could this model fit first, and what would we need to change in our Linux baseline, boot process, and attestation model to benefit from it?"

Good early candidates might be new platform builds, controlled edge environments, high-value workloads, or environments where kernel-level runtime controls and attestation already have a business case. For large legacy fleets, the path will be longer.

## A promising direction

I like the direction Shirudo is taking. The industry has spent years getting better at finding vulnerabilities, scanning artifacts, and generating advisories. We have spent less time improving the middle ground between "we know" and "we have fully patched". That middle ground is where incidents happen.

Shirudo does not remove the need for disciplined patch management, asset inventory, runtime detection, or secure engineering. It adds a possible new layer: signed, measurable, runtime shields that can be deployed faster than a reboot cycle. If AI keeps compressing the discovery timeline, defenders will need more tools in that layer.

I am looking forward to going deeper with my team later this year. If you want to understand the project beyond the announcement, I suggest starting with the [Isovalent lab](https://isovalent.com/labs/shirudo-getting-started/), reading the [Shirudo documentation](https://docs.shirudo.dev/), and checking the [GitHub repository](https://github.com/shirudo-dev/shirudo). There is also a virtual panel on December 15, 2026, [The Future of Self-Defending Systems: Deep Dive into Shirudo](https://isovalent.com/events/shirudo-virtual-panel/), which should be a good opportunity to hear directly from the people building and maintaining it.

## References

- [Shirudo GitHub repository](https://github.com/shirudo-dev/shirudo)
- [Shirudo documentation](https://docs.shirudo.dev/)
- [Shirudo system requirements](https://docs.shirudo.dev/operate/plan/system-requirements.html)
- [Shirudo threat model](https://docs.shirudo.dev/security/threat-model.html)
- [Isovalent lab: Getting Started with Shirudo](https://isovalent.com/labs/shirudo-getting-started/)
- [Isovalent event: The Future of Self-Defending Systems, Deep Dive into Shirudo](https://isovalent.com/events/shirudo-virtual-panel/)
- [Four Critical Flaws in Mature Software: How AI Is Changing Vulnerability Discovery](/posts/ai-changing-vulnerability-discovery-four-critical-flaws/)
