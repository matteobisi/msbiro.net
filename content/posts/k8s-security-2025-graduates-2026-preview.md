---
title: "Kubernetes Security 2025: Stable Features & 2026 preview"
date: 2025-12-08T10:05:05Z
tags: [
  "kubernetes", "cybersecurity", "devops", "devSecOps",
  "container-security", "cloud-native", "security-hardening", "supply-chain-security"
  ]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Recap of Kubernetes security features that reached stable in 2025 + predictions for 2026 graduates. DevSecOps guide to production hardening."
canonicalURL: "https://www.msbiro.net/posts/k8s-security-2025-graduates-2026-preview"
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

Like your favorite music streaming service's 2025 Wrapped®, here's my recap of Kubernetes security highlights from 2025—plus predictions for features likely graduating to stable in early 2026.
As a DevSecOps Team Leader, I bridge development speed with security rigor daily. Kubernetes and cloud-native security are my passion, especially hardening workloads for production.
With Kubernetes v1.35 releasing December 17, now's the perfect time to review 2025's security wins and plan for 2026.


Before we dive in, a quick reminder on how Kubernetes evolves(and unveil the way how I'll make my prediction later). The community officially supports the **last three releases**. Each new version brings features in different stages: `alpha` (experimental and requires manual enabling), `beta` (closer to stable and often enabled by default for testing), and `stable` (production-ready and reliable).  

Understanding this lifecycle is key to planning your security strategy.

---

### 2025 Kubernetes Security: Stable Graduates

2025 marked important progress in Kubernetes security, with key features graduating to stable status between versions 1.32 and 1.35. These advancements enhance authentication, authorization, workload isolation, and overall hardening for production cloud-native environments. Here are the most impactful stable graduates from 2025:

| Feature | Stable in | Description | Documentation |
| :--- | :--- | :--- | :--- |
| **Bound ServiceAccount token improvements** | v1.33 | Adds unique token IDs and node binding to service account tokens, improving validation, auditability, and limiting token reuse or node impersonation. | https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/ |
| **Sidecar Containers** | v1.33 | Native sidecar containers become a stable Pod lifecycle primitive, making it safer and more reliable to run security agents, proxies, and observability sidecars alongside workloads. | https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/ |
| **Recursive Read-Only (RRO) mounts** | v1.33 | Graduate to stable, allowing volumes to be mounted fully read-only (including subpaths), closing write paths that attackers could previously abuse on supposedly read-only mounts. | https://kubernetes.io/blog/2023/10/11/recursive-read-only-mounts-beta/ |
| **Finer-grained authorization using selectors** | v1.34 | Authorizers (built‑in and webhook) can now make decisions based on field/label selectors, so policies can restrict list/watch/deletecollection requests to, for example, only pods bound to a specific node. | https://kubernetes.io/docs/reference/access-authn-authz/authorization/ |
| **Restrict anonymous requests to specific endpoints** | v1.34 | Anonymous access can be limited to an explicit allowlist of endpoints (such as `/healthz`, `/readyz`, `/livez`), reducing the blast radius of RBAC misconfigurations that accidentally expose resources to unauthenticated users. | https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests |
| **Ordered namespace deletion** | v1.34 | Namespace deletion now follows a structured order so that pods are removed before dependent resources like NetworkPolicies, mitigating gaps where workloads could continue running without the intended network controls (e.g. CVE‑2024‑7598). | https://kubernetes.io/blog/2025/08/27/kubernetes-v1-34-release/ |

---

## The Future is Now: What to Expect in 2026

Looking at the alpha and beta features in Kubernetes 1.35 gives us a clear indication of what's likely to graduate to stable in 2026. These are the advancements that will further solidify Kubernetes as a secure platform. My team is already testing these in our labs, and I encourage you to do the same.

| Feature | Stage in v1.35 | KEP | Description | Documentation |
|---------|----------------|-----|-------------|---------------|
| Harden kubelet serving certificate validation | Alpha new in 1.35 | [KEP-4872](https://github.com/kubernetes/enhancements/issues/4872) | Adds an API-server check that the kubelet's serving certificate CN matches `system:node:<nodename>`, closing node-impersonation MitM attacks | [KEP-4872](https://github.com/kubernetes/enhancements/issues/4872) |
| Constrained impersonation | Alpha new in 1.35 | [KEP-5284](https://github.com/kubernetes/enhancements/issues/5284) | Tightens impersonate so that an impersonating user cannot perform actions they themselves are not allowed to do | [KEP-5284](https://github.com/kubernetes/enhancements/issues/5284) |
| User namespaces for HostNetwork Pods | Alpha new in 1.35 | [KEP-5607](https://github.com/kubernetes/enhancements/issues/5607) | Allows `hostNetwork: true` pods to keep `hostUsers: false` for better containment | [KEP-5607](https://github.com/kubernetes/enhancements/issues/5607) |
| CSI ServiceAccount tokens via Secrets | Alpha new in 1.35 | [KEP-5538](https://github.com/kubernetes/enhancements/issues/5538) | Moves CSI driver ServiceAccount tokens into dedicated secrets field | [KEP-5538](https://github.com/kubernetes/enhancements/issues/5538) |
| Robust image pull authorization | Beta new in 1.35 | [KEP-2535](https://github.com/kubernetes/enhancements/issues/2535) | Kubelet re-verifies registry credentials even for cached images | [KEP-2535](https://github.com/kubernetes/enhancements/issues/2535) |
| Pod certificates for mTLS | Beta promoted in 1.35 | [KEP-4317](https://github.com/kubernetes/enhancements/issues/4317) | PodCertificateRequest API for workload X.509 certificates | [KEP-4317](https://github.com/kubernetes/enhancements/issues/4317) |
| User namespaces for pods | Beta on-by-default in 1.35 | [KEP-127](https://github.com/kubernetes/enhancements/issues/127) | Pod-level user namespaces become more mature for multitenant isolation | [KEP-127](https://github.com/kubernetes/enhancements/issues/127) |
| Image volumes | Beta promoted in 1.35 | [KEP-4639](https://github.com/kubernetes/enhancements/issues/4639) | OCI images as readonly volumes with tighter registry policies | [KEP-4639](https://github.com/kubernetes/enhancements/issues/4639) |

---

### Conclusion

Kubernetes security continues to evolve rapidly, with v1.35, releasing soon, bringing [User Namespaces](https://www.msbiro.net/posts/kubernetes-133-user-namespace-isolation-security-matters/) to **beta on-by-default status**, one of my favorite hardening features for workload isolation.  
Looking ahead, I'm particularly excited for advancements in secrets management like Pod certificates for mTLS (beta in v1.35, KEP-4317) and robust image pull authorization (beta, KEP-2535) that promise tighter credential handling and workload identity.

Tracking the `alpha-to-stable` journey is a DevSecOps best practice—test these features in pre-production environments when relevant to your day-2 operations and stay ahead of emerging threats.
