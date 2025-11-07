---
title: "Runc Container Breakout Vulnerabilities"
date: 2025-11-07T06:45:00+00:00
tags: [
  "cloud-native", "kubernetes", "cybersecurity",
  "open-source", "devops", "containers", "runc"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A summary of the recently disclosed runc container breakout vulnerabilities (CVE-2025-31133, CVE-2025-52565, and CVE-2025-52881) and the recommended actions."
canonicalURL: "https://www.msbiro.net/posts/runc-container-breakout-vulnerabilities-2025/"
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
    alt: "Runc Container Breakout Vulnerabilities"
    caption: "Runc Container Breakout Vulnerabilities"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

On November 5th, 2025, a set of high-severity vulnerabilities in runc were publicly disclosed, allowing for full container breakouts. Runc is the cornerstone of containerization on Linux, serving as the default low-level container runtime for industry-standard tools like Docker, Podman, and Kubernetes. Its ubiquity means that a vulnerability in runc has far-reaching implications for the entire cloud-native ecosystem. This post summarizes the vulnerabilities, the affected versions, and the recommended actions to mitigate them.

## The Vulnerabilities

Three vulnerabilities were discovered, all related to bypassing runc's restrictions on writing to arbitrary `/proc` files. These vulnerabilities can be exploited by starting containers with custom mount configurations, including those defined in Dockerfiles (`RUN --mount=...`).

### CVE-2025-31133: "container escape via 'masked path' abuse due to mount race conditions"

- **CVSS:** 7.3 (CVSS:4.0/AV:L/AC:L/AT:P/PR:L/UI:A/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H)
- **Description:** This vulnerability exploits an issue with how masked paths are implemented. An attacker can replace `/dev/null` with a symlink to a `procfs` file (e.g., `/proc/sys/kernel/core_pattern` or `/proc/sysrq-trigger`). Runc will then bind-mount the symlink target read-write, leading to a container breakout or a host crash.

### CVE-2025-52565: "container escape with malicious config due to /dev/console mount and related races"

- **CVSS:** 7.3 (CVSS:4.0/AV:L/AC:L/AT:P/PR:L/UI:A/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H)
- **Description:** Similar to the previous CVE, this vulnerability exploits a flaw in `/dev/console` bind-mounts. By replacing `/dev/pts/$n` with a symlink, an attacker can get runc to bind-mount the symlink target over `/dev/console`, allowing read-write access to sensitive `procfs` files.

### CVE-2025-52881: "container escape and denial of service due to arbitrary write gadgets and procfs write redirects"

- **CVSS:** 7.3 (CVSS:4.0/AV:L/AC:L/AT:P/PR:L/UI:A/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H)
- **Description:** This is a more sophisticated vulnerability that bypasses LSM (Linux Security Modules) checks. An attacker can make `/proc/self/attr/<label>` reference a real `procfs` file, which can be used to redirect writes to malicious targets like `/proc/sysrq-trigger` (host crash) or `/proc/sys/kernel/core_pattern` (full container breakout).

## Exploitation Scenarios and Threat Model

These vulnerabilities could be abused in real-world setups where a user can run a container from a malicious or compromised image. For example, a multi-tenant environment where users can define their own containers could be at risk. An attacker could craft a Dockerfile with a malicious `RUN --mount=...` instruction that, when built and run, would trigger one of these vulnerabilities and allow them to escape the container and gain control of the underlying host.

## Kubernetes and Cloud-Native Implications

In a Kubernetes environment, the impact of these vulnerabilities is **magnified**.  
A successful exploit could allow an attacker to escape a container and potentially gain access to the node's kubelet, which would give them control over all other pods running on that node. From there, they could potentially compromise the entire cluster. This is why it is critical for all Kubernetes users to update their container runtimes as soon as possible.

## Affected Versions and Patches

Users are strongly recommended to update to one of the following runc releases as soon as possible:

*   runc v1.4.0-rc.3
*   runc v1.3.3
*   runc v1.2.8

## Mitigations

Besides upgrading runc, the following mitigations can be applied:

*   **User Namespaces:** Use [user namespaces](https://www.msbiro.net/posts/kubernetes-133-user-namespace-isolation-security-matters/) with the host root user not mapped into the container's namespace.
*   **Non-root user:** Do not run as a root user in the container.
*   **AppArmor/SELinux:** While not a complete solution, using AppArmor and SELinux can help to mitigate some of the attack vectors.

It's also important to note that the real-world impact of these vulnerabilities is significantly reduced in environments that follow security best practices. If you are using a controlled set of hardened and secured container images and you monitor your deployments with CI/CD pipelines for security and linting controls, the risk of exploitation is much lower. These vulnerabilities are most critical in scenarios where untrusted or unvetted container images can be introduced and run.

## The Bigger Picture: Secure-by-Default Configurations

These vulnerabilities highlight the need for secure-by-default configurations for container runtimes. The OpenSSF and the Open Container Initiative (OCI) are working on standardizing such configurations to reduce the attack surface and prevent similar vulnerabilities in the future. This is an ongoing effort, and it is crucial for the community to get involved and contribute to these initiatives.

## Other Container Runtimes

Other container runtimes like `youki` and `crun` are also understood to have similar flaws and are working on patches. Users of other runtimes should check for security updates from their vendors.

## Credits

Credits for the discovery of these vulnerabilities go to:

*   Lei Wang (@ssst0n3 from Huawei)
*   Li Fubang (@lifubang from acmcoder.com, CIIC)
*   Tõnis Tiigi (@tonistiigi from Docker)
*   Aleksa Sarai (@cyphar from SUSE)