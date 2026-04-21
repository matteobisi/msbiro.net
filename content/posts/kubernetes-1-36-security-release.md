---
title: "Kubernetes 1.36: The Release That Said Goodbye to Ingress NGINX"
date: 2026-04-21T12:35:00Z
tags: ["kubernetes", "security", "selinux", "ingress", "gateway-api", "cloud-native", "containers", "devops", "open-source"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Kubernetes 1.36 releases tomorrow with a significant security focus: the end of Ingress NGINX, SELinux volume labeling reaching GA, and a set of long-overdue removals that tighten the security posture of every cluster."
canonicalURL: "https://www.msbiro.net/posts/kubernetes-1-36-security-release/"
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
    alt: "Kubernetes 1.36 security release"
    caption: "Kubernetes 1.36 ships tomorrow — a release shaped by a commitment to a safer, more maintainable ecosystem."
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

Tomorrow, April 22, 2026, Kubernetes 1.36 will be officially released. As a team leader working in security, part of my job is reading release notes to understand what is coming and, more importantly, to track the direction the developers are moving in. Some releases are routine progress; others signal a shift in priorities. This is one of those.

Kubernetes 1.36 will be remembered as the release that formalized the end of Ingress NGINX. That alone would make it memorable; Ingress NGINX is too big and too deeply embedded in the ecosystem to ignore, and I will dedicate a section to it. But the focus of this post is security: alongside the NGINX retirement, 1.36 delivers meaningful hardening through the graduation of faster SELinux volume labeling to GA, the stable release of external ServiceAccount token signing, and the permanent removal of features that have been known security liabilities for years.

If security posture is part of your job, this is the release to read carefully.

## The End of Ingress NGINX

Let me be direct: as of March 24, 2026, Ingress NGINX is retired and will never receive another security patch.

On that date, Kubernetes SIG Network and the Security Response Committee announced the formal retirement of the `ingress-nginx` project. Starting from that date, there are no further releases, no bug fixes, and no updates to address security vulnerabilities. The GitHub repositories are now read-only. Existing deployments will continue to work, and Helm charts and container images remain available, but the project is effectively frozen in time.

This is a significant moment. Ingress NGINX became one of the most deployed pieces of infrastructure in the Kubernetes ecosystem. It was flexible, cloud-agnostic, and backed by a large user community. But that flexibility became its burden. The `snippets` annotation (once celebrated for allowing arbitrary NGINX configuration) became a recognized attack surface. CVEs kept accumulating. Maintainership gradually fell to a handful of individuals working on their own time. Even the attempt to build a successor project, InGate, never gained enough traction to become a viable replacement.

The Kubernetes project made the right call. Keeping a security-critical component alive with barely-sufficient maintenance is worse than acknowledging the reality: the project needs to end so users can migrate to something better.

### What should you migrate to?

The recommended path is [Gateway API](https://gateway-api.sigs.k8s.io/guides/), the modern replacement for the Ingress resource. Gateway API addresses the design limitations of the original Ingress API: it is expressive, role-oriented, and extensible by design. If your workload has more specific routing needs or you are not ready to fully migrate to Gateway API, there are many alternative Ingress controllers listed in the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

To check whether you are currently running Ingress NGINX in your cluster:

```bash
kubectl get pods --all-namespaces --selector app.kubernetes.io/name=ingress-nginx
```

If this returns any results, migration should be on your roadmap immediately.

## Security: What 1.36 Hardens

### Faster SELinux Labeling Reaches GA

One of the most impactful security improvements in this release is the graduation to stable of faster SELinux volume labeling ([KEP-1710](https://kep.k8s.io/1710)).

This feature has been in development since it first appeared as beta in Kubernetes 1.28 for `ReadWriteOncePod` volumes. The core problem it solves is real and painful: on SELinux-enforcing systems, when a Pod starts and mounts a volume, the kubelet traditionally relabels every file in the volume recursively to apply the correct SELinux context. For volumes with millions of files, this can mean Pod startup times measured in minutes, not seconds.

The fix is elegant: instead of walking the entire filesystem tree, Kubernetes now uses the `mount -o context=XYZ` option to apply the SELinux label to the volume at mount time as a whole. The result is consistent performance regardless of volume size, and a dramatically faster Pod startup on SELinux-enforcing nodes.

In 1.32, the feature gained metrics and an opt-out path via `securityContext.seLinuxChangePolicy: Recursive`, useful for catching conflicts during adoption. Now in 1.36, it becomes the default for all volumes, with Pods and CSIDrivers opting in via `spec.SELinuxMount`.

The important caveat here is that this feature shifts responsibility to Pod authors. If you have privileged and unprivileged Pods sharing volumes and you are not careful about `seLinuxChangePolicy` and SELinux volume labels, you will hit problems. This applies whether you are writing a Deployment, a StatefulSet, a DaemonSet, or a custom resource with a Pod template. The Kubernetes project is explicit about this: getting it wrong can cause subtle and hard-to-debug failures.

For security-conscious operators running SELinux-enforcing nodes (common in regulated industries and government environments), this is the most operationally meaningful change in 1.36.

### External Signing of ServiceAccount Tokens (GA)

Kubernetes 1.36 also brings the graduation to stable of external signing for ServiceAccount tokens ([KEP-740](https://kep.k8s.io/740)).

The feature allows the `kube-apiserver` to delegate token signing to an external system — a cloud KMS (Key Management Service), a hardware security module, or another centralized signing infrastructure — instead of relying exclusively on keys managed internally by the API server. In highly regulated environments, managing signing keys internally is often not acceptable: centralizing key management in a KMS allows organizations to enforce rotation policies, audit key usage, and integrate Kubernetes into their existing secrets management architecture without maintaining a parallel key system for the cluster. For teams already invested in solutions like AWS KMS, GCP Cloud KMS, or HashiCorp Vault, this removes a meaningful integration gap.

### `gitRepo` Volume Driver Removed

The `gitRepo` volume plugin has been deprecated since Kubernetes 1.11. In 1.36, it is permanently disabled and cannot be re-enabled ([KEP-5040](https://kep.k8s.io/5040)).

The reason is straightforward: this volume type allowed a malicious actor to run code as root on the underlying node. There is no safe way to use it, and alternatives (init containers, git-sync sidecars) have been the recommended approach for years. If any of your workloads still depend on `gitRepo` volumes, they will fail to schedule after upgrading to 1.36. Audit your workloads before upgrading.

### Deprecation of `service.spec.externalIPs`

The `externalIPs` field in Service specs is being deprecated in 1.36 and is targeted for removal in v1.43 ([KEP-5707](https://kep.k8s.io/5707)).

This field has been a documented security issue for years: it enables man-in-the-middle attacks on cluster traffic, as captured in [CVE-2020-8554](https://github.com/kubernetes/kubernetes/issues/97076). From 1.36 onward, using `externalIPs` in your Service definitions will produce deprecation warnings. The migration paths are: LoadBalancer Services for cloud-managed ingress, NodePort for simple port exposure, or Gateway API for flexible and secure external traffic handling.

## Security Checklist Before You Upgrade

Before rolling out 1.36, these are the security-specific things I would verify in any environment.

**Check for `gitRepo` volume usage.** These will break at scheduling time with no recovery path:

```bash
kubectl get pods --all-namespaces -o json | \
  jq '.items[] | select(.spec.volumes[]?.gitRepo != null) | .metadata | "\(.namespace)/\(.name)"'
```

**Audit `externalIPs` in Services.** They still work in 1.36 but you will start getting deprecation warnings, and more importantly they represent an active MitM risk that you should have cleaned up already:

```bash
kubectl get services --all-namespaces -o json | \
  jq '.items[] | select(.spec.externalIPs != null and (.spec.externalIPs | length) > 0) | "\(.metadata.namespace)/\(.metadata.name)"'
```

**Ingress NGINX users: you are now running an unpatched controller.** The retirement happened on March 24; if your cluster still runs `ingress-nginx` and a new CVE drops, there will be no fix coming. This is the most urgent security exposure in the ecosystem right now.

**SELinux operators: review your Pod security contexts before rolling out 1.36.** The new `seLinuxChangePolicy` behavior is the default for all volumes; misconfigured contexts on Pods sharing volumes can cause failures. Set it explicitly rather than relying on implicit behavior.

## Final Thoughts

Kubernetes 1.36 is not a flashy release full of exciting new alpha features. It is a release that made hard decisions: retiring a beloved but unmaintainable project, removing a feature that has been a CVE vector for fifteen years, and promoting security improvements that benefit real-world production environments running SELinux. This kind of release is what a mature, security-aware project looks like.

The retirement of Ingress NGINX is the moment the Kubernetes community chose ecosystem health over backward compatibility theater. The SELinux GA graduation is the kind of unglamorous, high-impact improvement that operators on regulated infrastructure have been waiting for. Together, they tell a consistent story: Kubernetes 1.36 prioritized security and long-term maintainability over feature count.

That is, honestly, exactly what you want from a platform you run in production.

### References

- [Kubernetes v1.36 Sneak Peek — kubernetes.io blog](https://kubernetes.io/blog/2026/03/30/kubernetes-v1-36-sneak-peek/)
- [Kubernetes 1.36 CHANGELOG — GitHub](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.36.md)
- [Ingress NGINX Retirement Announcement — kubernetes.io blog](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/)
- [KEP-1710: Speed up recursive SELinux label change](https://kep.k8s.io/1710)
- [KEP-740: External signing of ServiceAccount tokens](https://kep.k8s.io/740)
- [KEP-5040: Remove gitRepo volume driver](https://kep.k8s.io/5040)
- [KEP-5707: Deprecate service.spec.externalIPs](https://kep.k8s.io/5707)
- [Gateway API migration guide](https://gateway-api.sigs.k8s.io/guides/)
