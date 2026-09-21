---
title: "MicroVMs vs Containers for AI Agents: ECB Cybersecurity Guide"
date: 2026-09-21T06:29:45+01:00
tags: [
  "cloud-native", "cybersecurity", "open-source", "virtualization",
  "microvm", "firecracker", "kata-containers", "ai-agents",
  "container-security", "platform-engineering", "ecb", "banking-security",
  "operational-resilience"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "What the ECB's AI cybersecurity letter means for banks: microVMs, containers, Firecracker, and Kata Containers for isolating AI agents."
keywords: [
  "ECB AI cybersecurity threats", "AI-enabled cybersecurity threats banks",
  "microVMs vs containers for AI agents", "Firecracker vs Kata Containers",
  "AI agent sandboxing", "container isolation", "banking operational resilience"
]
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

AI agents have turned sandboxing from an engineering detail into a leadership decision. A capable agent can install tools, read source code, build experiments, retry failed approaches, and search for a path out of its environment for hours. The old question, "Do we have a sandbox?", is no longer enough.

Trail of Bits showed why in its [VM escape experiment with a cyber-capable AI agent](https://blog.trailofbits.com/2026/08/26/vms-wont-contain-cyber-capable-agents/). The agent repeatedly escaped a QEMU/KVM environment by combining known but unpatched host weaknesses, fixes not yet included in the distribution, and eventually new vulnerabilities. The lesson is not that virtual machines are useless. It is that no technical boundary provides complete containment on its own.

MicroVMs reduce the virtual hardware and management surface exposed to a workload. They make a sound isolation design smaller, faster to recreate, and easier to apply consistently. They are an important layer of defence, not a silver bullet.

For banks, the [ECB's July 2026 letter on AI-enabled cybersecurity threats](/posts/ecb-ai-enabled-cybersecurity-threats-letter/) puts this isolation decision in a supervisory context. This article turns that context into a practical choice for technology leaders running untrusted code, multi-tenant services, CI workloads, and increasingly capable AI agents. A practical Firecracker walkthrough is available in my public [technical notebook](https://github.com/matteobisi/technical_notebook/blob/main/runbooks/13_firecracker_microvm.md).

## What is a microVM?

A microVM is a lightweight virtual machine designed for a narrow workload model. Like a conventional VM, it uses hardware virtualization and has its own guest kernel, memory, process tree, and virtual devices. Unlike a general-purpose VM, it exposes only a deliberately small set of devices and capabilities.

The value is not that a microVM is less of a VM. It is that it deliberately does less.

| Decision point | Traditional VM | MicroVM | Container |
| --- | --- | --- | --- |
| Primary design goal | Broad operating system and hardware compatibility | High-volume, focused workload isolation | Package and run an application efficiently |
| Kernel | Its own guest kernel | Its own guest kernel | Shares the host kernel |
| Virtual hardware | Large and flexible device catalogue | Deliberately small device model | No emulated hardware |
| Isolation boundary | Hardware virtualization | Hardware virtualization with a narrower VMM surface | Linux namespaces, cgroups, and other kernel controls |
| Typical workload | Long-lived servers, desktops, and specialist operating systems | Functions, containers, build jobs, and short-lived services | Application services, batch jobs, and developer workloads |
| Operational model | Managed as an individual machine | Created, configured, observed, and removed through automation | Created and scheduled through a container runtime or orchestrator |
| Main security tradeoff | More features can mean more interfaces to secure | Fewer exposed interfaces, but still a guest kernel, hypervisor, and host | Efficient, but a kernel flaw can affect both workload and host |

The technology differences matter to engineers. The operational model and security tradeoff should matter to leadership. A microVM helps a platform team make isolation repeatable, but it does not make a workload trustworthy or remove the need for patching, network controls, credential handling, logging, and incident response.

## What AI changes in the control model

For financial institutions, this is more than a technical thought experiment. The ECB letter reaches the same conclusion from a supervisory perspective: AI increases the speed and scale at which familiar weaknesses are discovered and exploited. It does not prescribe a particular sandbox. Instead, it calls for faster basic hygiene, accountable ownership, better monitoring, tested recovery, and evidence that those controls work. The ECB asked significant institutions to turn that assessment into an action plan by 31 October 2026.

MicroVMs are a risk-reduction control, not a substitute for those measures. They reduce the exposed virtual-machine surface and make clean environments easier to recreate. They cannot compensate for an unpatched host, an overly permissive network path, unmanaged third-party software, or credentials that the workload should never have received.

The leadership question is whether the full execution environment assumes compromise:

1. Is the host disposable and free of production credentials or valuable personal data?
2. Is the guest exposed only to the files, network destinations, and APIs it actually needs?
3. Are the host kernel, VMM, runtime, and guest assets updated at the pace security fixes become available?
4. Can the workload be stopped, investigated, and recreated from a known state?
5. Is there another control if the virtualization boundary fails?

MicroVMs strengthen the containment boundary. The remaining controls determine how much a compromise can achieve.

## Firecracker and Kata Containers are not the same product

The Trail of Bits article mentioned earlier identifies Firecracker as a better-suited option for containing AI agents. That makes Firecracker a useful starting point, but it does not make Kata Containers a direct alternative. The projects operate at different layers of the stack and solve different parts of the isolation problem.

[Firecracker](https://firecracker-microvm.github.io/) is a virtual machine monitor, or VMM. It runs on Linux through KVM and uses a constrained virtual device model for high-density workloads. A platform team can use its local API directly or build a service around it. Firecracker also includes `jailer`, an additional userspace isolation layer for the VMM process.

[Kata Containers](https://github.com/kata-containers/kata-containers) is a container runtime platform. It lets OCI and Kubernetes workloads run inside lightweight VMs, while preserving familiar container lifecycle and orchestration interfaces. Kata has a runtime, an in-guest agent, and support for several hypervisors. Firecracker is one of the hypervisors Kata can use.

The useful comparison is therefore about the layer your organisation needs:

| Decision point | Firecracker | Kata Containers |
| --- | --- | --- |
| What it is | A Linux VMM for building and running microVMs | A runtime that runs container workloads inside lightweight VMs |
| Best fit | A team building a dedicated execution platform or a serverless-style service | A team that wants VM isolation while retaining Kubernetes and OCI workflows |
| Main integration work | Guest image lifecycle, networking, storage, policy, scheduling, observability, and API orchestration | RuntimeClass, container runtime, guest assets, storage, networking, and cluster operations |
| Hypervisor choice | Firecracker itself is the VMM | Supports Firecracker, QEMU, Cloud Hypervisor, Dragonball, and other supported back ends |
| Advanced hardware needs | Deliberately limited | Can select another supported hypervisor when requirements include GPUs or confidential computing |

There is no universal winner. Firecracker is the lower-level building block, while Kata is the higher-level runtime integration. A platform can use Kata with Firecracker underneath, or use Firecracker directly when it needs full control of the execution service.

## Enterprise use cases that justify a microVM decision

MicroVMs are most valuable when a business has workloads with different trust levels, short or replaceable lifecycles, and a credible need for a stronger boundary than a shared host kernel. The following use cases show where that tradeoff is most compelling.

### Multi-tenant services and serverless execution

A service that executes customer functions, plugins, templates, or data transformations must assume that one tenant's code can be hostile or simply defective. A microVM per execution environment can place a hardware-virtualization boundary between tenants while retaining an automated, high-density model.

For leaders, the important question is not which VMM is used. It is whether tenant code can access shared credentials, internal networks, host storage, or metadata services. A microVM boundary adds value only when those surrounding paths are constrained too.

### Kubernetes workloads with mixed trust levels

Most Kubernetes clusters run many applications behind a common container runtime and host kernel. That is efficient, but not every workload deserves the same trust level. CI runners, third-party batch jobs, regulated workloads, confidential-computing workloads, and customer-supplied code may justify a distinct runtime class backed by Kata Containers.

Kata preserves Kubernetes scheduling and deployment workflows while providing each pod, or defined workload group, with a VM boundary. This is often easier to govern than asking every application team to operate its own VM lifecycle. It still requires a clear policy for which workloads use the runtime, how images and guest assets are patched, and how networking is segmented.

### AI coding agents and autonomous task runners

An agent that can browse the web, clone repositories, execute tests, modify files, and use credentials should be treated as an untrusted operator with automation advantages. A microVM can be one layer of a safer design, especially when it is created for a task and destroyed afterwards.

The strongest design starts with a non-sensitive host. It provides a minimal workspace instead of a writable developer home directory, uses short-lived credentials or a proxy that injects credentials without exposing their values, and allows only the network destinations required for the task. The microVM makes that model practical, but does not compensate for mounting a developer's SSH keys or granting broad production access.

### Build and test isolation

Build systems routinely execute dependency scripts, test fixtures, and pull-request code. For high-risk repositories or external contributions, microVM-backed runners can reduce the blast radius compared with executing those actions directly on shared workers.

This is a business decision about risk concentration. A build fleet that holds cloud credentials, package-publishing permissions, and access to every internal network is a high-value target. Isolating the workload helps, but separating credentials and permissions is the more important control.

## The leadership decision: make isolation measurable

MicroVMs are not a reason to move every workload out of containers or traditional VMs. They are a strong option when code is untrusted, tenants must be separated, or a workload needs a stronger boundary without abandoning cloud-native automation.

The decision only delivers value when the controls around it are measurable:

| Control | What leadership should require evidence of |
| --- | --- |
| Host and workload separation | No production credentials, valuable data, or broad host mounts are available to hostile workloads |
| Boundary minimisation | Explicitly approved network destinations, APIs, and files, with no unnecessary host sockets or metadata access |
| Patch and rebuild discipline | Defined remediation targets for the host kernel, VMM, runtime, and guest assets; short-lived environments recreated from known images |
| Defence in depth | Network policy, least-privilege identity, supply-chain controls, monitoring, and human approval gates alongside virtualization |
| Recovery and accountability | A tested destroy-and-recreate process, an audit trail, and named owners for exceptions |

Choose Firecracker when the organisation needs to build the execution platform and can own guest images, networking, storage, and operations. Choose Kata Containers when it needs VM isolation within an established Kubernetes or OCI operating model. Keep traditional VMs when broad hardware support, long-lived operating systems, GPUs, or confidential-computing capabilities drive the requirement.

The goal is not a promise that compromise is impossible. It is a smaller attack surface, faster recovery, and less value exposed if one boundary fails.

## Further reading

- [Trail of Bits: VMs won't contain cyber-capable agents](https://blog.trailofbits.com/2026/08/26/vms-wont-contain-cyber-capable-agents/)
- [ECB on AI-enabled cybersecurity threats: what banks must do by October 2026](/posts/ecb-ai-enabled-cybersecurity-threats-letter/)
- [Firecracker project documentation](https://firecracker-microvm.github.io/)
- [Kata Containers architecture](https://github.com/kata-containers/kata-containers/blob/main/docs/design/architecture/README.md)
- [Kata Containers hypervisor guide](https://github.com/kata-containers/kata-containers/blob/main/docs/hypervisors.md)
- [Firecracker Linux runbook in my technical notebook](https://github.com/matteobisi/technical_notebook/blob/main/runbooks/13_firecracker_microvm.md)
