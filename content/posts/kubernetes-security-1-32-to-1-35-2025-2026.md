---
title: "From 1.32 to 1.35: Kubernetes Security Features That Matter for 2025–2026”
date: 2025-12-08T00:05:05Z
tags: [
  "kubernetes", "cybersecurity", "devops", "devSecOps",
  "container-security", "cloud-native", "security-hardening", "supply-chain-security"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A DevSecOps leader's perspective on Kubernetes 1.35, recapping the most important security features that became stable in 2025 and predicting what's next for 2026."
canonicalURL: "https://www.msbiro.net/posts/kubernetes-security-1-32-to-1-35-2025-2026/"
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
    alt: "Kubernetes 1.35 Security Features"
    caption: "2025 Security Advancements in Kubernetes"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Like your favorite music streaming service's personalized "wrapped 2025," I want to recap my preferred security enhancements released in 2025 and make some predictions about the first half of 2026 for security features that will probably graduate to stable in that period.

As a DevSecOps Team Leader, my day-to-day involves bridging the gap between development speed and security rigor. I'm passionate about Kubernetes and the open-source community, and a huge part of my focus is on securing cloud-native workloads. With Kubernetes 1.35 just around the corner, it’s a great time to reflect on the security advancements of 2025 and look ahead to what's coming.

Before we dive in, a quick reminder on how Kubernetes evolves. The community officially supports the **last three releases**. Each new version brings features in different stages: `alpha` (experimental and requires manual enabling), `beta` (closer to stable and often enabled by default for testing), and `stable` (production-ready and reliable). Understanding this lifecycle is key to planning your security strategy.

---

### 2025's Stable Security Wins: The Top 8 Pillars

2025 has been a landmark year for Kubernetes security, with several crucial features graduating to stable between versions 1.32 and 1.35. These aren't just minor updates; they are foundational pillars for building a more secure cloud-native environment. Here are the top 8 most impactful features that became stable in 2025:

| Feature | Stable in | Description | Documentation |
| :--- | :--- | :--- | :--- |
| **Bound ServiceAccount token improvements** | v1.33 | Adds unique token IDs and node binding to service account tokens, improving validation, auditability, and limiting token reuse or node impersonation. [web:40] | https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/ |
| **Sidecar Containers** | v1.33 | Native sidecar containers become a stable Pod lifecycle primitive, making it safer and more reliable to run security agents, proxies, and observability sidecars alongside workloads. [web:40][web:66] | https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/ |
| **Recursive Read-Only (RRO) mounts** | v1.33 | Graduate to stable, allowing volumes to be mounted fully read-only (including subpaths), closing write paths that attackers could previously abuse on supposedly read-only mounts. [web:40] | https://kubernetes.io/blog/2023/10/11/recursive-read-only-mounts-beta/ |
| **Structured Authentication Configuration** | v1.34 | The `AuthenticationConfiguration` API and `--authentication-config` file format are now GA, enabling file-based, CEL‑validated, dynamically reloadable API-server authentication config instead of scattered flags. [web:37] | https://kubernetes.io/docs/reference/access-authn-authz/authentication/#structured-authentication-configuration |
| **Finer-grained authorization using selectors** | v1.34 | Authorizers (built‑in and webhook) can now make decisions based on field/label selectors, so policies can restrict list/watch/deletecollection requests to, for example, only pods bound to a specific node. [web:37] | https://kubernetes.io/docs/reference/access-authn-authz/authorization/ |
| **Restrict anonymous requests to specific endpoints** | v1.34 | Anonymous access can be limited to an explicit allowlist of endpoints (such as `/healthz`, `/readyz`, `/livez`), reducing the blast radius of RBAC misconfigurations that accidentally expose resources to unauthenticated users. [web:37] | https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests |
| **Ordered namespace deletion** | v1.34 | Namespace deletion now follows a structured order so that pods are removed before dependent resources like NetworkPolicies, mitigating gaps where workloads could continue running without the intended network controls (e.g. CVE‑2024‑7598). [web:37] | https://kubernetes.io/blog/2025/08/27/kubernetes-v1-34-release/ |
| **AppArmor support** | v1.34 | AppArmor support graduates to stable, letting clusters rely on kernel‑level mandatory access control profiles as a first‑class, officially supported hardening mechanism for Linux workloads. [web:37] | https://kubernetes.io/docs/tutorials/security/apparmor/ |


---

### The Future is Now: What to Expect in 2026

Looking at the `alpha` and `beta` features in Kubernetes 1.35 gives us a clear indication of what's likely to graduate to stable in 2026. These are the advancements that will further solidify Kubernetes as a secure platform. My team is already testing these in our labs, and I encourage you to do the same. Here are some of the most promising security features currently in development:

| Feature | Stage in v1.35 | Description | Documentation |
| :--- | :--- | :--- | :--- |
| **Harden kubelet serving certificate validation (KubeletCertCNValidation)** | Alpha (new in 1.35) [web:65] | Adds an API-server check that the kubelet’s serving certificate CN matches `system:node:<nodename>`, closing a class of node‑impersonation / MitM attacks when node IPs are reused. [web:65] | https://github.com/kubernetes/enhancements/issues/4872 |
| **Constrained impersonation** | Alpha (new in 1.35) [web:65] | Tightens `impersonate` so that an impersonating user cannot perform actions they themselves are not allowed to do, reducing the risk of over‑privileged debug or proxy workflows. [web:65] | https://github.com/kubernetes/enhancements/issues/5284 |
| **User namespaces for HostNetwork Pods (UserNamespacesHostNetworkSupport)** | Alpha (new in 1.35) [web:65] | Allows `hostNetwork: true` pods to keep `hostUsers: false`, so workloads can access the host network stack without also gaining host‑user privileges, improving containment if the pod is compromised. [web:65] | https://github.com/kubernetes/enhancements/issues/5607 |
| **CSI ServiceAccount tokens via Secrets (CSIServiceAccountTokenSecrets)** | Alpha (new in 1.35) [web:65] | Moves CSI driver ServiceAccount tokens out of `volumeContext` into a dedicated `secrets` field, separating sensitive credentials from non‑sensitive metadata and reducing accidental leakage. [web:65] | https://github.com/kubernetes/enhancements/issues/5538 |
| **Robust image pull authorization (KubeletEnsureSecretPulledImages)** | Beta (new in 1.35) [web:65] | Introduces `imagePullCredentialsVerificationPolicy` so kubelet can re‑verify registry credentials even when images are already cached, closing scenarios where pods could reuse pre‑pulled images without proper auth. [web:65] | https://github.com/kubernetes/enhancements/issues/2535 |
| **Pod certificates for mTLS (PodCertificateRequest)** | Beta (promoted in 1.35) [web:65] | The `PodCertificateRequest` API and `PodCertificate` volume source move to beta, making it easier to issue workload X.509 certificates for first‑class mTLS between pods and the API server or other services. [web:65] | https://github.com/kubernetes/enhancements/issues/4317 |
| **User namespaces for pods (UserNamespacesSupport)** | Beta (on-by-default in 1.35) [web:65][web:40] | Pod‑level user namespaces, previously alpha/beta and opt‑in, become a more mature hardening option for running “root in the pod, unprivileged on the host”, improving multi‑tenant isolation. [web:65][web:40] | https://github.com/kubernetes/enhancements/issues/127 |
| **Image volumes (VolumeSource: OCI Artifact/ImageVolume)** | Beta (promoted in 1.35) [web:65] | Using OCI images as read‑only volumes is now beta, enabling separation of binaries and content; from a security POV, it demands tighter policies around which registries and images may be mounted into pods. [web:65][web:37] | https://github.com/kubernetes/enhancements/issues/4639 |


---

### Conclusion

Kubernetes is constantly evolving, and its security capabilities are getting stronger with every release. The journey from `alpha` to `stable` is our roadmap for the future. By understanding this roadmap, we can build more secure systems and make life harder for attackers.

As members of the open-source community, sharing our experiences is vital. What are your security priorities for Kubernetes in 2026? Are you testing any of these upcoming features? I'd love to hear your thoughts.
