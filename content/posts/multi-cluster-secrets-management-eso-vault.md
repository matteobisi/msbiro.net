---
title: "Centralized Secrets Management for Multi-Cluster Kubernetes: An Architecture Pattern"
date: 2026-01-14T23:20:00+00:00
tags: [
  "kubernetes", "secrets-management", "external-secrets-operator",
  "hashicorp-vault", "openbao", "devsecops", "cloud-native", "architecture"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A production-grade guide to implementing centralized secrets management across multiple Kubernetes clusters using External Secrets Operator and HashiCorp Vault. Includes architecture diagrams, Vault policy configuration, and advanced patterns."
canonicalURL: "https://www.msbiro.net/posts/multi-cluster-secrets-management-eso-vault/"
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
    alt: "Multi-cluster secrets management architecture"
    caption: "Centralized vs Siloed Secrets Management"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

As organizations scale their Kubernetes footprint, moving from a single cluster to a multi-cluster architecture (often spanning Dev, Staging, and Production), the challenge of managing sensitive information—API keys, database credentials, and certificates—grows exponentially.

In this post, we'll go beyond the basics and explore a **production-grade architecture** for centralized secrets management using **External Secrets Operator (ESO)** and **HashiCorp Vault** (or **OpenBao**).

---

### The Architecture: Hub-and-Spoke

The most effective pattern for multi-cluster environments is a "Hub-and-Spoke" model.

*   **The Hub:** A centralized, highly available Vault cluster. This is your "Source of Truth" where secrets are rotated, audited, and governed.
*   **The Spokes:** Multiple Kubernetes clusters (Dev, QA, Prod), each running the External Secrets Operator.

{{< mermaid >}}
graph TD
    subgraph "Central Security Zone"
        Vault[HashiCorp Vault / OpenBao]
        Policy[("ACL Policies")]
    end

    subgraph "Production Cluster (Spoke)"
        ESO_Prod[External Secrets Operator]
        Secret_Prod[K8s Secret: db-pass]
        App_Prod[Production App]
    end

    subgraph "Dev Cluster (Spoke)"
        ESO_Dev[External Secrets Operator]
        Secret_Dev[K8s Secret: db-pass]
        App_Dev[Dev App]
    end

    ESO_Prod -- "Auth: K8s (Prod Role)" --> Vault
    ESO_Dev -- "Auth: K8s (Dev Role)" --> Vault
    Vault -- "Inject Secret (Prod)" --> ESO_Prod
    Vault -- "Inject Secret (Dev)" --> ESO_Dev
    ESO_Prod --> Secret_Prod
    Secret_Prod --> App_Prod
{{< /mermaid >}}

### Step 1: Configuring the Trust (Vault Side)

The critical piece often missing from tutorials is the **handshake**. How does your central Vault trust a remote Kubernetes cluster? You need to configure the **Kubernetes Auth Method** for *each* cluster you want to connect.

**On your Vault server:**

1.  Enable the Auth Method for the specific cluster (e.g., `kubernetes-prod`):

    ```bash
    vault auth enable -path=kubernetes-prod kubernetes
    ```

2.  Configure the connection details. You'll need the Host URL and CA Certificate of your *Production Kubernetes API Server*.

    ```bash
    vault write auth/kubernetes-prod/config \
        kubernetes_host="https://api.k8s-prod.example.com" \
        kubernetes_ca_cert="@/path/to/k8s-prod-ca.crt" \
        issuer="https://kubernetes.default.svc.cluster.local"
    ```

    > **Pro Tip:** Ensure your Vault has network reachability to the K8s API server. If they are in different VPCs, you might need VNet Peering or a Transit Gateway.

### Step 2: Defining Granular Policies

Security implies isolation. A compromised Dev cluster should *never* be able to read Production secrets. We enforce this with Vault Policies.

**Create a policy for the Dev environment:**

```hcl
# policy-dev.hcl
path "secret/data/dev/*" {
  capabilities = ["read"]
}

# Deny access to everything else explicitly (default behavior, but good for clarity)
path "secret/data/prod/*" {
  capabilities = ["deny"]
}
```

**Bind this policy to a Role:**

This tells Vault: "If a ServiceAccount named `eso-sa` from the `external-secrets` namespace in the *Dev Cluster* logs in, give them the `dev-policy`."

```bash
vault write auth/kubernetes-dev/role/eso-dev-role \
    bound_service_account_names=eso-sa \
    bound_service_account_namespaces=external-secrets \
    policies=dev-policy \
    ttl=1h
```

### Step 3: The Operator Configuration (K8s Side)

Now we move to the Kubernetes cluster. We use a `ClusterSecretStore` to define the connection once for the entire cluster.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: central-vault
spec:
  provider:
    vault:
      server: "https://vault.corp.example.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          # IMPORTANT: matches the path we enabled in Step 1
          mountPath: "kubernetes-dev" 
          role: "eso-dev-role"
          serviceAccountRef:
            name: "eso-sa"
            namespace: "external-secrets"
```

### Advanced Patterns

#### 1. Handling Rotation & Restarts
Vault rotates database credentials automatically. ESO picks up the change based on its `refreshInterval` (e.g., every 5 minutes) and updates the native K8s Secret.

However, most applications read secrets only at startup. To complete the rotation, you need to restart the application pods. The community standard for this is **Reloader**.

Add the annotation to your Deployment:

```yaml
metadata:
  annotations:
    reloader.stakater.com/auto: "true"
```

Now, when ESO updates the secret, Reloader detects the change and performs a rolling restart of your pods.

#### 2. Templating for Usability
Sometimes the raw data in Vault doesn't match what your application expects. ESO has a powerful templating engine.

**Scenario:** Vault has `username` and `password` as separate keys, but your Java app needs a connection string.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: jdbc-string
spec:
  target:
    name: app-db-secret
    template:
      data:
        # Create a new key 'url' by combining inputs
        jdbc_url: "jdbc:postgresql://db.example.com:5432/mydb?user={{ .username }}&password={{ .password }}"
  data:
  - secretKey: username
    remoteRef:
      key: secret/data/dev/db
      property: username
  - secretKey: password
    remoteRef:
      key: secret/data/dev/db
      property: password
```

### Resilience: What if Vault goes down?

One of the biggest advantages of this pattern over "Sidecar Injection" (like Vault Agent) is **resilience**.

Because ESO synchronizes the secret into a *native Kubernetes Secret*, your application depends only on the Kubernetes API during startup. If the central Vault goes down for maintenance, your applications **continue to run and scale**, consuming the last known good secret cached in Etcd.

### Conclusion

By combining **HashiCorp Vault's** robust policy engine with **External Secrets Operator's** Kubernetes-native design, you achieve a sweet spot: centralized governance with decentralized resilience.

Are you using this pattern? Or do you prefer the Sidecar approach? Let's discuss on [LinkedIn](https://www.linkedin.com/in/matteobisi/).