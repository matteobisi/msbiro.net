---
title: "Kubernetes 2025 Security: A DevSecOps Leader's Recap of my preferred feature and anticipation for 2026"
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
canonicalURL: "https://www.msbiro.net/posts/kubernetes-1-35-security-recap-2025/"
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
| **Structured Authorization Configuration** | v1.32 | Provides a more granular and structured way to manage RBAC policies. | [Link](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) |
| **Bound Service Account Token Improvements** | v1.32 | Enhances security by providing a unique identifier for each token, improving traceability. | [Link](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/) |
| **Pod Security Standards (PSS)** | v1.32 | Provides a clear, out-of-the-box mechanism for enforcing pod security policies. | [Link](https://kubernetes.io/docs/concepts/security/pod-security-admission/) |
| **Immutable Secrets and ConfigMaps** | v1.32 | Prevents accidental or malicious modifications and improves performance. | [Link](https://kubernetes.io/docs/concepts/configuration/secret/#secret-immutability) |
| **Sidecar Containers** | v1.33 | Provides a more robust and integrated way to deploy security agents and other auxiliary containers. | [Link](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) |
| **User Namespace Isolation** | v1.33 | Isolates pod user namespaces from the host, improving workload isolation and security. | [Link](https://kubernetes.io/docs/concepts/workloads/pods/user-namespaces/) |
| **Structured Authentication Configuration** | v1.34 | Allows for file-based configuration of API server authenticators with CEL validation and dynamic reloading. | [Link](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#structured-authentication-configuration) |
| **Finer-Grained Authorization & Anonymous Controls** | v1.34 | Adds field/label selectors for more precise list/watch decisions and provides endpoint-specific anonymous request limits. | [Link](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) |

---

### The Future is Now: Predictions for Stable Security Features in 2026

Looking at the `beta` features in Kubernetes 1.35 gives us a clear indication of what's likely to graduate to stable in the first half of 2026. These are the advancements that will further solidify Kubernetes as a secure platform. My team is already testing these in our labs, and I encourage you to do the same. Even further out, `alpha` features hint at a more dynamic and intelligent Kubernetes. Think **context-aware security policies** that adapt to real-time threats, or deeper integration with **hardware security modules (HSMs)**. This is the exciting frontier, where Kubernetes becomes not just a container orchestrator, but a truly security-aware platform.

#### Prediction 1: Supply Chain Security will be Natively Enforced

`Beta` features for verifying container image signatures are maturing fast. I predict that by mid-2026, **image signature validation will be a stable, out-of-the-box capability**. This means we'll be able to create policies that prevent unsigned or untrusted images from ever running in our clusters—a massive step forward for supply chain security.

#### Prediction 2: Secrets Management Gets a Boost

While existing secrets management is functional, new `beta` features are set to improve how secrets are stored and accessed, especially with external secret providers. I foresee a future where Kubernetes offers more native, secure, and seamless integration with tools like **HashiCorp Vault or AWS Secrets Manager**, reducing operational overhead for everyone.

---

### Conclusion

Kubernetes is constantly evolving, and its security capabilities are getting stronger with every release. The journey from `alpha` to `stable` is our roadmap for the future. By understanding this roadmap, we can build more secure systems and make life harder for attackers.

As members of the open-source community, sharing our experiences is vital. What are your security priorities for Kubernetes in 2026? Are you testing any of these upcoming features? I'd love to hear your thoughts.
