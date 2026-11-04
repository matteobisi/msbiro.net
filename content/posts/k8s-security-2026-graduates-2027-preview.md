---
title: "Kubernetes Security: 2026 Stable Features & 2027 preview"
date: 2026-12-16T09:00:00Z
tags: ["kubernetes", "cybersecurity", "devsecops", "container-security", "cloud-native"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Every Kubernetes security feature that reached stable in 2026, scored against last year's predictions, plus the alpha and beta work that should land in 2027."
canonicalURL: "https://www.msbiro.net/posts/k8s-security-2026-graduates-2027-preview"
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
    alt: "Kubernetes 1.38 Security Features"
    caption: "2026 Security Advancements in Kubernetes"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Last December I wrote a recap of the Kubernetes security features that reached stable in 2025, plus a table of predictions for 2026. A year has passed, three minor releases shipped, and v1.38 landed on December 16. Time to check the score and do it again.

Writing predictions is easy. Publishing the results is the part that keeps you honest, so this year I start with the scoreboard before proposing anything new.

The setup is the same as last time. Kubernetes supports the last three releases. Each new version carries features at different stages: `alpha` (experimental, must be enabled), `beta` (closer to stable, often on by default), and `stable` (production ready, with API and behaviour guarantees). Tracking that progression is how you plan a security roadmap instead of reacting to release notes.

One note on scope. I am looking at security relevant work only: authentication, authorization, admission, credential handling, workload isolation, and the hardening primitives around them.

---

## Scorecard: how the 2025 predictions landed

Eight features in last year's table. Four reached stable, two advanced a stage without finishing, two did not move.

| Feature | KEP | Predicted in v1.35 | Outcome |
| :--- | :--- | :--- | :--- |
| CSI driver SA tokens via secrets | [KEP-5538](https://github.com/kubernetes/enhancements/issues/5538) | Alpha new | **GA in v1.36** |
| User namespaces for Pods | [KEP-127](https://github.com/kubernetes/enhancements/issues/127) | Beta on by default | **GA in v1.36** |
| Image volumes | [KEP-4639](https://github.com/kubernetes/enhancements/issues/4639) | Beta promoted | **Stable in v1.36** |
| Pod certificates for mTLS | [KEP-4317](https://github.com/kubernetes/enhancements/issues/4317) | Beta promoted | **GA in v1.37** |
| Constrained impersonation | [KEP-5284](https://github.com/kubernetes/enhancements/issues/5284) | Alpha new | Beta in v1.36, not GA yet |
| User namespaces for HostNetwork Pods | [KEP-5607](https://github.com/kubernetes/enhancements/issues/5607) | Alpha new | Beta, gated behind NodeDeclaredFeatures |
| Robust image pull authorization | [KEP-2535](https://github.com/kubernetes/enhancements/issues/2535) | Beta new | Still Beta |
| Harden kubelet serving certificate validation | [KEP-4872](https://github.com/kubernetes/enhancements/issues/4872) | Alpha new | Still Alpha, marked stale |

Four out of eight is a decent hit rate for a one year horizon in Kubernetes, and the two misses that advanced to Beta are the normal case: a feature spends a release at Beta collecting field feedback before it is allowed to graduate.

The two that did not move are worth naming. Robust image pull authorization (KEP-2535) is the one I would most like to see finished, because the gap it closes is real: a kubelet that trusts credentials stored at pull time keeps working from cache when the credential is later revoked. KEP-4872 sits at Alpha and carries a `lifecycle/stale` label, which in practice means nobody is actively pushing it. If it is still there next year I will say so again, more loudly.

---

## 2026 stable graduates

Between v1.36 in April and v1.38 in December, these are the security relevant features that reached stable.

| Feature | Stable in | Why it matters |
| :--- | :--- | :--- |
| **Mutating admission policies** ([KEP-3962](https://github.com/kubernetes/enhancements/issues/3962)) | v1.36 | CEL based mutating policy without running a webhook. Removes a whole class of "the webhook was down, so the mutation did not apply" failures. |
| **Fine grained kubelet API authorization** ([KEP-2862](https://github.com/kubernetes/enhancements/issues/2862)) | v1.36 | Authorize requests to the kubelet API individually instead of a flat allow or deny. The kubelet stops being an all or nothing backdoor for anyone with node level reach. |
| **External ServiceAccount token signer** | v1.36 | The API server can delegate token signing to an external component, so keys leave the control plane process. |
| **CSI ServiceAccount token secrets** ([KEP-5538](https://github.com/kubernetes/enhancements/issues/5538)) | v1.36 | Tokens requested by CSI drivers move into a dedicated secrets field, so a driver that only needs a token cannot read unrelated secrets. |
| **SELinux mount policies** ([KEP-1710](https://github.com/kubernetes/enhancements/issues/1710)) | v1.36 | SELinux labelling for volumes happens at mount time on read write once pods, cutting the recursive relabel step that made SELinux expensive on large volumes. |
| **User namespaces for Pods** ([KEP-127](https://github.com/kubernetes/enhancements/issues/127)) | v1.36 | Root inside the container is no longer root on the node. This is the single biggest isolation win of the year and it took years to land. |
| **Image volumes** ([KEP-4639](https://github.com/kubernetes/enhancements/issues/4639)) | v1.36 | OCI images mounted as read only volumes, so artefacts and rule sets arrive through the same registry controls as container images. |
| **Pod certificates** ([KEP-4317](https://github.com/kubernetes/enhancements/issues/4317)) | v1.37 | PodCertificateRequest gives workloads short lived X.509 identities from the API server. No more long lived secrets mounted into a pod just to speak mTLS. |
| **SELinux mount** ([KEP-1710](https://github.com/kubernetes/enhancements/issues/1710)) | v1.37 | Enabled by default, and this one broke clusters. Read the [release post](https://kubernetes.io/blog/2026/04/22/breaking-changes-in-selinux-volume-labeling/) before upgrading if you run SELinux. |
| **DRA device taints and tolerations** ([KEP-5055](https://github.com/kubernetes/enhancements/issues/5055)) | v1.37 | Devices can be marked tainted and Pods must tolerate them, the same model as node taints but for accelerators and other DRA resources. |
| **DRA device taint rules** | v1.38 | Admin driven tainting of devices by selector, with the feature gates locked to enabled. Both gates are scheduled for removal in v1.41. |
| **WebSocket streaming authorization** | v1.38 | The gate that requires create permission on the Pod for a WebSocket upgrade is now permanently on, alongside the port forward and stream translation gates. |

Two entries deserve more than a table row.

**User namespaces reaching GA** is the headline of 2026 in my book. It changes the threat model for anything running as root in a container. The practical consequence is that a container escape no longer starts from a uid that maps to something privileged on the host. If you have not tested it, the upgrade path is unglamorous: check that your nodes have subordinate ID ranges configured in `/etc/subuid` and `/etc/subgid`, because v1.38 tightened validation there and a bad range now stops the kubelet from starting.

**Pod certificates reaching GA, then getting post quantum support at Beta in v1.38**, is the other one. Workload identity issued by the control plane, with ML-DSA key types accepted behind a feature gate. That combination is where certificate management is going.

---

## What to expect in 2027

These are the alpha and beta features shipping today that I expect to move next year. Same caveat as always: a stage is not a promise.

| Feature | Stage now | What it changes |
| :--- | :--- | :--- |
| **Pod certificates with ML-DSA keys** | Beta in v1.38 | Post quantum workload identities from the PodCertificateRequest API, with MLDSA44, MLDSA65 and MLDSA87 key types. |
| **OCI based security profile distribution** ([KEP-6061](https://github.com/kubernetes/enhancements/issues/6061)) | Alpha in v1.38 | Seccomp profiles pulled from OCI registries through the CRI, rather than copied onto every node. Security profiles become versioned artefacts with digests. |
| **Conditional authorization** ([KEP-5681](https://github.com/kubernetes/enhancements/issues/5681)) | Alpha | Authorization decisions that carry conditions, so a policy can express "allowed, but only under these constraints" instead of a binary yes or no. |
| **Manifest based admission control** ([KEP-5793](https://github.com/kubernetes/enhancements/issues/5793)) | Beta, on by default | Admission webhooks and CEL policies loaded from static manifests on disk. A compromised API server cannot delete its own admission controls through the API. |
| **Constrained impersonation** ([KEP-5284](https://github.com/kubernetes/enhancements/issues/5284)) | Beta | An impersonating user can no longer do things they could not do themselves. |
| **User namespaces for HostNetwork Pods** ([KEP-5607](https://github.com/kubernetes/enhancements/issues/5607)) | Beta | Keeps `hostUsers: false` available for pods that need the host network, which was the last big hole in the user namespace story. |
| **Scoped tokens for admission webhooks** | Alpha in v1.37 | API servers authenticate to admission webhooks with scoped service account tokens instead of shared credentials. |
| **Robust image pull authorization** ([KEP-2535](https://github.com/kubernetes/enhancements/issues/2535)) | Beta | Kubelet re-verifies registry credentials for images already in cache. |

If I have to pick the one to watch, it is OCI based security profile distribution. Seccomp profiles are already the most underused hardening control in Kubernetes, and the reason is distribution, not capability. Nobody enjoys maintaining a DaemonSet that copies JSON files onto nodes. Pulling them from a registry like any other artefact removes that friction at the root.

---

## Where I would spend my test hours

You cannot evaluate everything. In my lab the order would be:

1. **User namespaces on a realistic workload**, including a stateful one. The ID range details are where this breaks, not the concept.
2. **Pod certificates as a replacement for mounted mTLS secrets.** Start with one internal service mesh hop before promising anything.
3. **Mutating admission policies**, replacing the simplest webhook you run. The interesting question is not whether it works, but whether you can now audit what your admission chain actually enforces.
4. **Manifest based admission control**, if you run clusters where the API server is the thing that might be compromised. That is the threat model it was designed for.

Everything else can wait for the release notes to age a version.

---

## Conclusion

2026 was the year workload isolation stopped being an add on in Kubernetes. User namespaces went GA, fine grained kubelet authorization went GA, and pod certificates went from beta to GA with post quantum key types already at Beta behind them. That is a coherent direction: remove the shared, long lived credential, and remove the assumption that root in a container is fine.

2027 looks like the year of distribution and conditions. Security profiles moving into registries, admission control moving onto disk, authorization gaining conditions, identity gaining post quantum algorithms. Less glamorous than isolation, and closer to where production incidents actually come from.

Last year I said that tracking the alpha to stable journey is a DevSecOps practice, not a hobby. A year later, with four of eight predictions landed and two features still stuck, I would put it slightly differently: track it, then write down what you predicted, because the places where Kubernetes does not move tell you more about your roadmap than the places where it does.
