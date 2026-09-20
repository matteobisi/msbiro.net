---
title: "AI Agents Broke the Old Sandbox Model: MicroVMs vs Containers"
date: 2026-09-19T17:29:45+01:00
tags: [
  "cloud-native", "cybersecurity", "open-source", "virtualization",
  "microvm", "firecracker", "kata-containers", "ai-agents",
  "container-security", "platform-engineering"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "AI agents changed the isolation decision. An executive guide to traditional VMs, microVMs, containers, Firecracker, and Kata Containers for secure enterprise workloads."
canonicalURL: "https://www.msbiro.net/posts/microvms-vs-containers-ai-security/"
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
    alt: "MicroVM isolation boundary for enterprise workloads"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

AI agents have turned sandboxing from an engineering detail into a leadership decision. A capable agent can install tools, read source code, build experiments, retry failed approaches, and keep searching for a path out of its environment for hours. The old question, "Do we have a sandbox?", is no longer enough.

Trail of Bits made this concrete in its [VM escape experiment with a cyber-capable AI agent](https://blog.trailofbits.com/2026/08/26/vms-wont-contain-cyber-capable-agents/). The agent escaped a QEMU/KVM environment repeatedly by combining known but unpatched host weaknesses, fixes that had not reached the distribution, and eventually new vulnerabilities. The lesson is not that virtual machines are useless. It is that no single technical boundary is a complete containment strategy.

MicroVMs are relevant because they reduce the virtual hardware and management surface exposed to a workload. They can make a sound isolation design smaller, faster to recreate, and easier to apply consistently. They are not a silver bullet.

This article is aimed at technology leaders deciding how to run untrusted code, multi-tenant services, CI workloads, or increasingly capable AI agents. The practical Firecracker walkthrough is maintained separately in my public [technical notebook](https://github.com/matteobisi/technical_notebook/blob/main/runbooks/13_firecracker_microvm.md).

## What is a microVM?

A microVM is a lightweight virtual machine designed for a narrow workload model. Like a conventional VM, it uses hardware virtualization and has its own guest kernel, memory, process tree, and virtual devices. Unlike a general-purpose VM, it intentionally exposes a small set of devices and capabilities.

The value is not that a microVM is somehow less of a VM. The value is that it does less.

| Decision point | Traditional VM | MicroVM | Container |
| --- | --- | --- | --- |
| Primary design goal | Broad operating system and hardware compatibility | High-volume, focused workload isolation | Package and run an application efficiently |
| Kernel | Its own guest kernel | Its own guest kernel | Shares the host kernel |
| Virtual hardware | Large and flexible device catalogue | Deliberately small device model | No emulated hardware |
| Isolation boundary | Hardware virtualization | Hardware virtualization with a narrower VMM surface | Linux namespaces, cgroups, and other kernel controls |
| Typical workload | Long-lived servers, desktops, and specialist operating systems | Functions, containers, build jobs, and short-lived services | Application services, batch jobs, and developer workloads |
| Operational model | Managed as an individual machine | Created, configured, observed, and removed through automation | Created and scheduled through a container runtime or orchestrator |
| Main security tradeoff | More features can mean more interfaces to secure | Fewer exposed interfaces, but still a guest kernel, hypervisor, and host | Efficient, but a kernel flaw can affect both workload and host |

The first three differences matter to engineers. The last two should matter to leadership. A microVM helps a platform team make isolation repeatable. It does not make the workload trustworthy and it does not remove the need for patch management, network control, credential handling, logging, or incident response.

## The control problem AI exposes

For financial institutions, this is more than a technical thought experiment. The [ECB's July 2026 letter on AI-enabled cybersecurity threats](/posts/ecb-ai-enabled-cybersecurity-threats-letter/) makes the same point from a supervisory perspective: AI increases the speed and scale at which familiar weaknesses are discovered and exploited. Its answer is not a mandate to adopt a particular sandbox. It is faster basic hygiene, accountable ownership, better monitoring, tested recovery, and evidence that those controls work. The ECB asked significant institutions to turn that assessment into an action plan by 31 October 2026.

MicroVMs fit this agenda as a risk-reduction control, not as a substitute for it. They can reduce the exposed virtual-machine surface and make a clean environment easier to recreate. They cannot compensate for an unpatched host, an overly permissive network path, unmanaged third-party software, or credentials that should never have been available to the workload.

The leadership question is whether the entire execution environment is designed for compromise:

1. Is the host disposable and free of production credentials or valuable personal data?
2. Is the guest exposed only to the files, network destinations, and APIs it actually needs?
3. Are the host kernel, VMM, runtime, and guest assets updated at the pace security fixes become available?
4. Can the workload be stopped, investigated, and recreated from a known state?
5. Is there another control if the virtualization boundary fails?

MicroVMs improve the answer to some of these questions. They do not answer all of them.

## Firecracker and Kata Containers are not the same product

It is tempting to compare Firecracker and Kata Containers as two competing microVM products. That is useful at the budget-planning level, but technically incomplete.

[Firecracker](https://firecracker-microvm.github.io/) is a virtual machine monitor, or VMM. It runs on Linux through KVM and is designed with a constrained virtual device model for high-density workloads. A platform team can use its local API directly, or build a service around it. Firecracker also includes `jailer`, an additional userspace isolation layer for the VMM process.

[Kata Containers](https://github.com/kata-containers/kata-containers) is a container runtime platform. It lets OCI and Kubernetes workloads run inside lightweight VMs, while preserving familiar container lifecycle and orchestration interfaces. Kata has a runtime, an in-guest agent, and support for several hypervisors. Firecracker is one of the hypervisors Kata can use.

That means the practical comparison is about the layer your organisation needs:

| Decision point | Firecracker | Kata Containers |
| --- | --- | --- |
| What it is | A Linux VMM for building and running microVMs | A runtime that runs container workloads inside lightweight VMs |
| Best fit | A team building a dedicated execution platform or a serverless-style service | A team that wants VM isolation while retaining Kubernetes and OCI workflows |
| Main integration work | Guest image lifecycle, networking, storage, policy, scheduling, observability, and API orchestration | RuntimeClass, container runtime, guest assets, storage, networking, and cluster operations |
| Hypervisor choice | Firecracker itself is the VMM | Supports Firecracker, QEMU, Cloud Hypervisor, Dragonball, and other supported back ends |
| Advanced hardware needs | Deliberately limited | Can select another supported hypervisor when requirements include GPUs or confidential computing |

Neither row declares a universal winner. Firecracker is the lower-level building block. Kata is the higher-level runtime integration. A platform may use Kata with Firecracker underneath, or use Firecracker directly where it needs full control of the execution service.

## Enterprise use cases that justify a microVM decision

MicroVMs are most valuable when the business has many workloads with different trust levels, short or replaceable lifecycles, and a credible need for a stronger boundary than a shared host kernel.

### Multi-tenant services and serverless execution

A service that executes customer functions, plugins, templates, or data transformations must assume that one tenant's code can be hostile or simply defective. A microVM per execution environment can place a hardware-virtualization boundary between tenants while retaining an automated, high-density model.

For leaders, the important decision is not the brand of VMM. It is whether tenant code can access shared credentials, internal networks, host storage, or metadata services. A microVM boundary is valuable only when those surrounding paths are also constrained.

### Kubernetes workloads with mixed trust levels

Most Kubernetes clusters run many applications behind a common container runtime and host kernel. That is efficient, but not every workload deserves the same trust level. CI runners, third-party batch jobs, regulated workloads, and customer-supplied code may justify a distinct runtime class backed by Kata Containers.

Kata can preserve Kubernetes scheduling and deployment workflows while providing each pod, or a defined workload group, with a VM boundary. This is often easier to govern than asking every application team to operate its own VM lifecycle. It still requires a clear policy for which workloads receive the runtime, how images and guest assets are patched, and how networking is segmented.

### AI coding agents and autonomous task runners

An agent that can browse the web, clone repositories, execute tests, modify files, and use credentials should be handled as an untrusted operator with automation advantages. A microVM can be one layer of a safer design, especially if it is created per task and destroyed afterward.

The strongest design starts with a non-sensitive host. It provides a minimal workspace rather than a writable developer home directory, uses short-lived credentials or a proxy that injects credentials without exposing their values, and allows only the network destinations required for the task. The microVM makes that model practical. It does not compensate for mounting a developer's SSH keys or granting broad production access.

### Build and test isolation

Build systems routinely execute dependency scripts, test fixtures, and pull-request code. For high-risk repositories or external contributions, microVM-backed runners can reduce the blast radius compared with executing those actions directly on shared workers.

This is a business decision about risk concentration. A company that has one build fleet holding cloud credentials, package-publishing permissions, and access to every internal network is creating a high-value target. Isolating the workload is useful, but separating the credentials and permissions is the more important control.

## The leadership decision: make isolation measurable

MicroVMs are not a reason to move every workload out of containers or traditional VMs. They are a strong option where code is untrusted, tenants must be separated, or a workload needs a stronger boundary without abandoning cloud-native automation.

The decision only delivers value when the controls around it are measurable:

| Control | What leadership should require evidence of |
| --- | --- |
| Host and workload separation | No production credentials, valuable data, or broad host mounts are available to hostile workloads |
| Boundary minimisation | Explicitly approved network destinations, APIs, and files, with no unnecessary host sockets or metadata access |
| Patch and rebuild discipline | Defined remediation targets for the host kernel, VMM, runtime, and guest assets; short-lived environments recreated from known images |
| Defence in depth | Network policy, least-privilege identity, supply-chain controls, monitoring, and human approval gates alongside virtualization |
| Recovery and accountability | A tested destroy-and-recreate process, an audit trail, and named owners for exceptions |

Choose Firecracker when the organisation needs to build the execution platform and can own its guest images, networking, storage, and operations. Choose Kata Containers when the organisation needs VM isolation inside an established Kubernetes or OCI operating model. Keep traditional VMs where broad hardware support, long-lived operating systems, GPUs, or confidential-computing capabilities drive the requirement.

The goal is not a promise that compromise is impossible. It is a smaller attack surface, faster recovery, and less value exposed if one boundary fails.

## Further reading

- [Trail of Bits: VMs won't contain cyber-capable agents](https://blog.trailofbits.com/2026/08/26/vms-wont-contain-cyber-capable-agents/)
- [ECB on AI-enabled cybersecurity threats: what banks must do by October 2026](/posts/ecb-ai-enabled-cybersecurity-threats-letter/)
- [Firecracker project documentation](https://firecracker-microvm.github.io/)
- [Kata Containers architecture](https://github.com/kata-containers/kata-containers/blob/main/docs/design/architecture/README.md)
- [Kata Containers hypervisor guide](https://github.com/kata-containers/kata-containers/blob/main/docs/hypervisors.md)
- [Firecracker Linux runbook in my technical notebook](https://github.com/matteobisi/technical_notebook/blob/main/runbooks/13_firecracker_microvm.md)
