---
title: "Ubuntu 26.04 LTS: What Changes for Security and Container Workloads"
date: 2026-04-29T08:00:00+01:00
tags: [
  "ubuntu", "linux", "cloud-native", "kubernetes", "containers",
  "cybersecurity", "devops", "golden-master", "cgroup", "post-quantum"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Ubuntu 26.04 LTS 'Resolute Raccoon' just shipped. For teams running RHEL or Ubuntu on servers, this post breaks down what actually changed in security and container/Kubernetes workloads compared to 24.04 LTS, and whether it justifies starting the golden master rebuild now."
canonicalURL: "https://www.msbiro.net/posts/ubuntu-2604-lts-security-upgrade-golden-master/"
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
    alt: "Ubuntu 26.04 LTS security and container changes"
    caption: "Ubuntu 26.04 LTS security and container changes"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Ubuntu 26.04 LTS ("Resolute Raccoon") shipped on April 24, 2026. Most of the coverage has focused on the desktop and the new Security Center UI, but I work almost exclusively on the server and infrastructure side, so I want to look at what actually matters for the teams I work with: those running Ubuntu Server as a base for VMs, bare-metal nodes, Kubernetes workers, and golden master images.

My customers are split between RHEL and Ubuntu. The ones on Ubuntu are typically on 22.04 (few) or 24.04 LTS (most). The question I always get after a new LTS is the same: "Do we need to move now, or can we sit on the current version for another year?" This post is my attempt to give a structured answer, focused on security and container workloads, which is where I can actually add value.

---

## Security on Bare-Metal Servers

**Kernel 7.0** is the base for everything else in this section. The jump from kernel 6.8 (Ubuntu 24.04) to 7.0 brings several changes relevant to physical hardware and to the general OS hardening baseline.

**Shadow Stack (Intel CET)** is a hardware-enforced control-flow integrity mechanism that prevents return-oriented programming (ROP) attacks by maintaining a separate, read-only shadow call stack. On modern Intel CPUs (Ice Lake and later) or AMD (Zen 3 and later), your processes get this protection automatically without any application recompilation. For servers running long-lived Java or Go services, this is free hardening that requires no configuration.

**Kernel Lockdown Mode** ships in integrity mode by default. It prevents the root user from modifying the running kernel through interfaces like `/dev/mem`, unsigned modules, or certain debugfs paths. For environments where post-compromise persistence is a concern, this closes a well-known class of techniques attackers use to maintain access after gaining root.

**TPM-backed Full Disk Encryption** is now natively supported at install time. The encryption key is sealed to the physical TPM against the system's PCR measurements, so the server boots and decrypts without manual passphrase input, but only if nothing in the boot chain has changed. For data center servers and headless deployments, this removes the traditional trade-off between automated boot and disk encryption. Verify that your provisioning tooling supports the necessary installer parameters; the [Ubuntu autoinstall documentation](https://ubuntu.com/server/docs/install/autoinstall) covers the relevant options.

**sudo-rs** replaces the default `sudo` binary. It is a complete [rewrite in Rust](https://github.com/trifectatechfoundation/sudo-rs) that eliminates the memory-safety class of bugs at the source, the same class that produced Baron Samedit (CVE-2021-3156) and similar privilege escalation CVEs in the original C implementation. sudo-rs is a drop-in replacement and maintains compatibility with `/etc/sudoers`. Test any custom sudoers configurations, particularly complex `NOPASSWD` rules or custom plugins, before deploying.

**AppArmor 5.x** ships with broader default profiles in enforce mode. In 24.04, several system services ran in complain mode or had incomplete profiles. In 26.04, the coverage is wider. If you have custom AppArmor profiles that reference cgroup v1 paths or legacy `docker-default` profile patterns, they need to be reviewed before you migrate. In March 2026, Qualys disclosed a set of AppArmor vulnerabilities affecting multiple LTS versions including 26.04. Canonical patched them promptly via the [Ubuntu Security Notices](https://ubuntu.com/security/notices) (USN) process. Keeping unattended-upgrades enabled and monitoring your USN feed remains non-negotiable regardless of OS version.

**Post-quantum SSH** is enabled by default in [OpenSSH 10.2](https://www.openssh.com/releasenotes.html). New SSH connections use a hybrid ML-KEM (formerly Kyber) combined with classical ECDH for key exchange, giving you quantum-resistant key agreement without any configuration. This addresses harvest-now-decrypt-later attacks: adversaries collecting encrypted SSH sessions today to decrypt them once quantum computing matures. One caveat: some older SSH clients embedded in legacy automation tooling may have compatibility issues with the hybrid algorithms. Test your pipelines before assuming everything just works.

---

## Security on Virtual Machines

Everything in the previous section applies to VMs as well. What 26.04 adds specifically for the VM layer is production-ready confidential computing support.

**Intel TDX and AMD SEV-SNP** support is now mainlined in kernel 7.0. In 24.04, both were technically available but required patches or custom kernel builds in most setups. In 26.04 it works out of the box. [Intel TDX (Trust Domain Extensions)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html) creates hardware-isolated VMs where the hypervisor has no access to the guest's memory. [AMD SEV-SNP](https://www.amd.com/en/developer/sev.html) provides encrypted, integrity-protected VM memory that the host cannot read or tamper with.

The practical path to using these features in container workloads is [Kata Containers](https://katacontainers.io/): each container runs inside a lightweight, hardware-isolated VM backed by TDX or SEV-SNP. From a Kubernetes perspective the interface is familiar: you add a `RuntimeClass` pointing to the `kata-qemu-tdx` or `kata-qemu-snp` handler and containerd manages the lifecycle. The kernel 7.0 support means you no longer need to maintain a custom kernel package to make this work. For multi-tenant workloads or environments handling regulated data in shared infrastructure, the barrier to adopting confidential containers dropped considerably with this release.

Shadow Stack also works in VMs if the hypervisor exposes the CPU virtualization extensions for CET (VMX CET support for Intel, SVM for AMD). Most modern KVM setups on patched kernels do, but verify your hypervisor version and guest CPU configuration before relying on it.

---

## Container and Kubernetes: The Breaking Change You Cannot Miss

### cgroup v2 Exclusive: No More v1

cgroup v2 has been the default since Ubuntu 22.04 and Kubernetes 1.25 (GA, August 2022). By 2026 it is already the standard for any modernly configured cluster, and Kubernetes upstream put cgroup v1 in maintenance mode with 1.31. What changes in Ubuntu 26.04 is not cgroup v2 itself — it is that the v1 fallback is gone entirely. If your workloads run correctly on 22.04 or 24.04 with cgroup v2, there is nothing to do here. The only real concern is a containerd config still explicitly set to `SystemdCgroup = false`, or legacy automation that hardcodes cgroup v1 paths (`/sys/fs/cgroup/memory/`, `/sys/fs/cgroup/cpu/`) — both of which should have been cleaned up years ago but are worth a quick check if you are migrating older infrastructure.

### containerd 1.7+ and OCI Runtime Changes

Ubuntu 26.04 ships containerd 1.7 as the default. The security-relevant changes are:

- Native support for OCI image layout improvements, including support for image encryption primitives that pair with the confidential computing features described above.
- The `runc` version bundled with containerd has been updated to include the fixes for CVE-2025-31133, CVE-2025-52565, and CVE-2025-52881 ([the container breakout vulnerabilities I wrote about in November 2025](https://www.msbiro.net/posts/runc-container-breakout-vulnerabilities-2025/)). If you have been patching these manually on 24.04, on 26.04 the patched version is the baseline.
- Improved rootless container support, which matters if your security posture requires running containers without the container daemon holding elevated privileges.

### Kubernetes Node Considerations

If you are building Kubernetes worker nodes on top of Ubuntu 26.04, two configuration items need attention: set `cgroupDriver: systemd` explicitly in the kubelet configuration and remove any `--cgroup-driver=cgroupfs` flags from kubelet startup arguments if they exist. Do not rely on autodetection.

Kernel 7.0 also ships with eBPF enhancements worth knowing about. If you run eBPF-based network policy or observability (Cilium, Tetragon, Falco with eBPF backend), expect better stability and fewer workarounds with no configuration change needed.

---

## Should You Build the New Golden Master Now?

If your current base is **Ubuntu 22.04 LTS**, start planning now. Standard support ends in April 2027 and building a new golden master around 24.04 at this point is a short-lived investment. Target 26.04 directly.

If you are on **Ubuntu 24.04 LTS**, there is no security emergency. Standard security support runs until April 2029 and none of the 26.04 improvements represent a vulnerability you are currently exposed to. Use 26.04 for new infrastructure from mid-2026 onward and plan migration of existing nodes in a controlled window.

In both cases: **wait for 26.04.1** (expected August 2026) before baselining production golden masters. The first point release consolidates the accumulated patches since launch and is the right baseline for stable, long-lived images.

Ubuntu 26.04 LTS is a meaningful release: post-quantum SSH by default, sudo rewritten in Rust, hardware-backed VM isolation, cgroup v2 exclusive, and kernel 7.0 with Shadow Stack and confidential computing on a stable LTS baseline.  
The migration is justified; the timeline depends on your team's capacity.

