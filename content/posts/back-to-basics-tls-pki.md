---
title: "Back to Basics: TLS and PKI from the Ground Up"
date: 2026-06-29T07:49:50+01:00
tags: [
  "cybersecurity", "devops", "linux", "open-source", "tutorial",
  "tls", "pki", "kubernetes", "devsecops", "certificates", "openssl",
  "x509", "mtls", "certificate-authority"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "TLS and PKI explained from the ground up: what an X.509 certificate actually contains, how the chain of trust works, what happens during a TLS handshake step by step, and how Kubernetes builds a full PKI with kubeadm that most engineers never read. Practical openssl commands throughout."
canonicalURL: "https://www.msbiro.net/posts/back-to-basics-tls-pki/"
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
    alt: "TLS PKI certificate chain diagram showing Root CA, Intermediate CA, and leaf certificate hierarchy"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

This is the third article in my "Back to Basics" series. The goal is simple: take something modern engineers interact with daily through abstractions, and explain what is actually happening underneath. In the [first article](https://www.msbiro.net/posts/back-to-basics-sshd-hardening/), I hardened an SSH daemon and explained why the defaults are insecure. In the [second](https://www.msbiro.net/posts/back-to-basics-containers-linux-processes/), I showed that containers are ordinary Linux processes wrapped in namespaces and cgroups. This article applies the same approach to TLS: strip away the abstractions, read the raw structures, and understand what the tooling is doing on your behalf.

I have been working with certificates for more than a decade, starting long before Kubernetes existed, manually concatenating PEM files and debugging handshake failures with `openssl s_client` at 2am because an intermediate CA was missing from the chain. The tooling has improved enormously since then, but the fundamentals have not changed at all. That is what makes them worth learning once and keeping forever.

Open a terminal, follow the commands as you read, and by the end you will be able to read a certificate by hand, trace a complete chain of trust, watch a TLS handshake happen live, and navigate the Kubernetes PKI without treating it as magic.

---

## The Foundation: Asymmetric Cryptography

TLS is built on asymmetric cryptography, which means every participant holds two mathematically linked keys: a private key and a public key. The relationship between them is the entire foundation of the system.

The **private key** must never leave the machine that generated it. It has two powers: it can decrypt data that was encrypted with the matching public key, and it can create a digital signature that anyone with the public key can verify. Keeping it secret is not optional; it is the single point of failure for the entire trust model.

The **public key** is designed to be shared with anyone. It can encrypt data that only the private key can decrypt, and it can verify signatures that only the private key could have produced. The math (RSA or elliptic curve) ensures these properties hold, and it also ensures that you cannot reverse-engineer the private key from the public key in any reasonable amount of time.

Generate a key pair and inspect it:

```bash
# Generate a 4096-bit RSA private key
openssl genrsa -out private.key 4096

# Extract the corresponding public key
openssl rsa -in private.key -pubout -out public.key

# Inspect the private key structure
openssl rsa -in private.key -text -noout | head -20
```

The output shows the modulus, the public exponent (always 65537 in practice), and the private exponent. You do not need to understand the math, but you should recognize that the private key file contains *both* keys. `openssl rsa -pubout` extracts just the public portion.

For new systems, prefer elliptic curve keys over RSA. They are shorter, faster, and offer equivalent security with smaller key sizes:

```bash
# Generate an EC private key using the P-256 curve
openssl ecparam -name prime256v1 -genkey -noout -out ec-private.key

# Extract the public key
openssl ec -in ec-private.key -pubout -out ec-public.key
```

---

## What Is an X.509 Certificate?

A certificate is a public key bundled with an identity claim and a third-party signature vouching for that claim. That is all it is. The identity claim says "this public key belongs to `api.example.com`," and the signature says "the issuer checked this and agrees."

The standard format is X.509, defined in RFC 5280. Every certificate contains:

- A **Subject** (who this cert is for): `CN=api.example.com`
- A **Subject Alternative Name** (SAN): the actual list of hostnames/IPs the cert is valid for; the CN is largely ignored by modern clients
- An **Issuer** (who signed it): `CN=My Intermediate CA`
- A **Validity period**: `notBefore` and `notAfter` timestamps
- The **Subject's public key**
- The **Issuer's digital signature** over all of the above

Generate a self-signed certificate and read it in full:

```bash
# Generate a self-signed cert (private key + cert in one command)
openssl req -x509 -newkey rsa:4096 -keyout self-signed.key \
  -out self-signed.crt -days 365 -nodes \
  -subj "/CN=example.local" \
  -addext "subjectAltName=DNS:example.local,DNS:localhost"

# Read every field
openssl x509 -in self-signed.crt -text -noout
```

Run it and look at these specific fields. Under `Validity`, the `Not After` date is the expiry; this is what cert-manager watches and what `kubeadm certs check-expiration` reports. Under `X509v3 Subject Alternative Name`, you will find the SANs; TLS clients match the hostname they are connecting to against *this* list, not the CN. If the SAN list is wrong or missing, the handshake will fail regardless of what the CN says.

The **issuer signature** is the critical field. For a self-signed cert, the issuer and subject are identical; the certificate signed itself. For any cert issued by a real CA, the issuer's name will differ and you need the issuer's certificate to verify the signature.

---

## The Chain of Trust

Self-signed certificates present a fundamental problem: anyone can generate one claiming to be `api.bank.com`. There is no way to distinguish a legitimate cert from a forged one unless you have a prior relationship with the entity that signed it.

The solution is a hierarchy. At the top sits a **Root CA**, a certificate authority whose public key is pre-installed in your operating system or browser. Root CAs do not sign end-entity certificates directly; they sign **Intermediate CAs**, which perform the day-to-day work of issuing certificates. This limits exposure: if an intermediate CA is compromised, it can be revoked without touching the root.

{{< mermaid >}}
graph TD
    A["🔐 Root CA<br/><small>Trusted by OS / browser trust store</small>"]
    B["🏛️ Intermediate CA<br/><small>Signed by Root, day-to-day issuance</small>"]
    C["🌐 api.example.com<br/><small>Leaf certificate, signed by Intermediate</small>"]
    A -->|signs| B
    B -->|signs| C
{{< /mermaid >}}


When your client receives the leaf certificate for `api.example.com`, it verifies the entire chain: it checks that the leaf cert's issuer signature is valid using the intermediate's public key, then checks that the intermediate's issuer signature is valid using the root's public key. The root is trusted because it is in the system trust store. If any link is broken, verification fails.

You can verify a chain manually:

```bash
# Download a site's certificate chain
openssl s_client -connect example.com:443 -showcerts 2>/dev/null \
  | awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/' > chain.pem

# Verify the leaf cert against the full chain
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt chain.pem
```

You can also inspect each certificate in the chain individually:

```bash
# Split a chain file into individual certs and inspect the first one
csplit -z -f cert- chain.pem '/-----BEGIN CERTIFICATE-----/' '{*}'
openssl x509 -in cert-00 -text -noout | grep -E "Subject:|Issuer:|Not After"
```

This chain verification is also exactly what happens when a Java application throws `PKIX path building failed: unable to find valid certification path to requested target`. The JVM ships with its own trust store (`cacerts`), separate from the OS, and it does not know about your internal CA. The fix is not to disable certificate validation. The fix is to import the internal root CA certificate into the JVM trust store:

```bash
# Import an internal CA certificate into the JVM trust store
keytool -import -trustcacerts -alias internal-ca \
  -file internal-ca.crt \
  -keystore $JAVA_HOME/lib/security/cacerts \
  -storepass changeit

# Verify it was imported
keytool -list -keystore $JAVA_HOME/lib/security/cacerts \
  -storepass changeit | grep internal-ca
```

The same logic applies to any tool with its own trust store: `curl` and `git` use the OS store, but the JVM, Python's `certifi` bundle, and Node.js each maintain their own. When a service refuses to connect to something protected by an internal cert, the question is always: which trust store is this client reading, and does it contain the signing CA?

---

## The TLS Handshake, Step by Step

Understanding the chain of trust makes the TLS handshake readable. What follows is TLS 1.3, the current standard.

### TLS 1.3 vs TLS 1.2

TLS 1.2 is still widely deployed and supported, but TLS 1.3 is the version you should be targeting. The most important practical difference is the number of round trips required to establish a connection:

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Handshake round trips | 2 | 1 (0-RTT for resumption) |
| Cipher negotiation | Client and server negotiate after hello | Client sends key exchange parameters in ClientHello |
| Forward secrecy | Optional | Mandatory |
| Deprecated algorithms | RC4, SHA-1, CBC ciphers still possible | Removed entirely |

This is why you commonly see `ssl_protocols TLSv1.2 TLSv1.3;` in Nginx configs: you keep 1.2 as a fallback for older clients while preferring 1.3 for everything that supports it. TLS 1.0 and 1.1 are deprecated and should be disabled everywhere.

### The Handshake Steps

The handshake establishes three things: mutual agreement on cryptographic algorithms, verification of the server's identity, and generation of shared session keys that neither side knew in advance.

**Step 1: ClientHello.** The client sends the TLS versions it supports, a list of cipher suites it accepts, and a random value (`client_random`). In TLS 1.3, the client also sends key exchange parameters immediately.

**Step 2: ServerHello.** The server selects the TLS version and cipher suite, sends its own random value (`server_random`), and returns its key exchange parameters.

**Step 3: Server Certificate.** The server sends its certificate chain, starting from the leaf and ending at the intermediate (the root is not sent because the client already has it in the trust store).

**Step 4: Client Verification.** The client verifies: (a) the signature chain is valid, (b) the `notBefore`/`notAfter` dates include today, and (c) the hostname it is connecting to matches one of the certificate's SANs. If any check fails, the handshake aborts here.

**Step 5: Key Derivation.** Both sides use the exchanged key material and their random values to independently derive the same session keys. No key is ever transmitted. An eavesdropper who captured the entire handshake cannot compute the session keys without the private key material that never left the server.

**Step 6: Finished.** Both sides exchange a `Finished` message, encrypted with the new session keys, proving that both possess the correct keys. Application data flows from this point.

Watch this happen live:

```bash
# Full handshake trace with timing
openssl s_client -connect github.com:443 -tls1_3 -msg 2>&1 | head -60

# Just the certificate details
openssl s_client -connect github.com:443 2>/dev/null | openssl x509 -text -noout

# Check expiry date for any hostname
echo | openssl s_client -connect github.com:443 2>/dev/null \
  | openssl x509 -noout -dates
```

`curl -v` is equally useful and more readable for humans:

```bash
curl -v https://github.com 2>&1 | grep -E "SSL|TLS|expire|issuer|subject"
```

---

## Certificate Signing Requests

When you need a certificate signed by a real CA rather than self-signed, you generate a **Certificate Signing Request** (CSR). The CSR contains your public key and the identity information you want on the certificate. You send it to the CA, which verifies your identity and returns a signed certificate. Your private key never leaves your machine.

```bash
# Generate a private key
openssl genrsa -out server.key 4096

# Generate a CSR with SANs
openssl req -new -key server.key -out server.csr \
  -subj "/CN=api.example.com" \
  -addext "subjectAltName=DNS:api.example.com,DNS:www.example.com"

# Inspect the CSR before sending it
openssl req -in server.csr -text -noout
```

This is exactly what cert-manager does automatically inside Kubernetes. When you create a `Certificate` resource, the controller generates a key, builds a CSR, submits it to the configured `Issuer` or `ClusterIssuer` (Let's Encrypt, Vault, or an internal CA), receives the signed certificate, and stores both the key and the cert in a `Secret`. The Secret is then mounted into your pod or referenced by an Ingress controller.

Follow each step as it happens:

```bash
# Watch cert-manager issue a certificate
kubectl describe certificate my-cert -n my-namespace

# The resulting Secret contains three keys
kubectl get secret my-cert-tls -n my-namespace -o jsonpath='{.data}' | jq 'keys'
# ["ca.crt", "tls.crt", "tls.key"]

# Inspect the issued certificate
kubectl get secret my-cert-tls -n my-namespace \
  -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text -noout
```

---

## Kubernetes PKI: Your Cluster Is a CA

`kubeadm init` does not just start Kubernetes. It generates an entire PKI from scratch, stored under `/etc/kubernetes/pki/`. Most engineers have never looked at this directory.

```bash
ls -la /etc/kubernetes/pki/
```

The directory contains three separate certificate authorities:

- `ca.crt` / `ca.key`: The **Kubernetes CA**, used to issue certificates for the API server, controller manager, scheduler, and kubelets.
- `etcd/ca.crt` / `etcd/ca.key`: The **etcd CA**, used exclusively for etcd client and peer communication.
- `front-proxy-ca.crt` / `front-proxy-ca.key`: The **front-proxy CA**, used for API server extension aggregation.

Each component in the cluster holds a certificate signed by the appropriate CA. The kubelet on each node presents a client certificate when it communicates with the API server. The API server presents a server certificate when you run `kubectl`. Every component mutually authenticates using this PKI.

Check the expiry of every certificate in the cluster:

```bash
kubeadm certs check-expiration
```

The output shows each certificate, how long it has left, and the CA that issued it. By default, kubeadm-issued certificates expire after one year. The cluster CA itself expires after ten years. When the cluster CA expires, every component stops trusting every other component simultaneously — the cluster goes dark.

Inspect the certificate your `kubectl` client uses to authenticate:

```bash
# Decode the client cert from kubeconfig
kubectl config view --raw \
  -o jsonpath='{.users[0].user.client-certificate-data}' \
  | base64 -d | openssl x509 -text -noout
```

Inspect the cluster CA:

```bash
# Decode the cluster CA from kubeconfig
kubectl config view --raw \
  -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' \
  | base64 -d | openssl x509 -text -noout
```

Renew all certificates before they expire (requires running this on the control plane node):

```bash
kubeadm certs renew all
```

---

## mTLS: Mutual TLS and Workload Identity

Standard TLS is one-sided: the server proves its identity to the client, but the client proves nothing. **Mutual TLS** (mTLS) requires both parties to present a valid certificate. The handshake is identical, with an additional step where the server requests the client's certificate and verifies it against a trusted CA.

This is how a service mesh like Istio secures pod-to-pod communication. The Istio sidecar injected into each pod:

1. Generates a private key locally.
2. Creates a CSR containing the pod's identity, encoded as a **SPIFFE ID** (a URI of the form `spiffe://cluster.local/ns/default/sa/my-service-account`).
3. Submits the CSR to `istiod`, the mesh control plane.
4. Receives back a short-lived X.509 certificate (a **SVID**, SPIFFE Verifiable Identity Document), typically valid for 24 hours.
5. Intercepts all inbound and outbound TCP traffic from the pod and presents this certificate during every connection.

The result: every connection between pods in the mesh is mutually authenticated and encrypted, and no application code changed. The identity proof is the workload's service account, not a hostname.

You can verify the certificate a pod is presenting without any Istio tooling:

```bash
# Get the mTLS cert being served by a pod's sidecar (port 15006 is Istio's inbound)
kubectl exec -n my-namespace my-pod -c istio-proxy -- \
  openssl s_client -connect localhost:15006 2>/dev/null \
  | openssl x509 -text -noout | grep -A3 "Subject Alternative Name"
# The SAN will show the SPIFFE URI, e.g.:
# URI:spiffe://cluster.local/ns/my-namespace/sa/my-service-account
```

SPIFFE and SVID are not new technology. They are a standardization of X.509 applied to workload identity, the same certificate format used everywhere else, with a URI-based naming convention that suits dynamic infrastructure better than static hostnames.

mTLS was something I understood theoretically for years, but never applied in a real production environment until my current role. It was only when I started working deeply with secrets management that I encountered [SPIFFE and SPIRE](https://spiffe.io/) in practice. Watching a workload receive a short-lived certificate, prove its identity to another service, and have the whole thing rotate transparently every 24 hours without a single manual step, that is when the engineering behind it clicked for me. It solves a genuinely hard problem in a clean way.

---

## What Goes Wrong (and How to Debug It)

**Self-signed certificates in production.** The temptation is to use `--insecure-skip-tls-verify` or equivalent flags to bypass certificate errors. This defeats the entire purpose of TLS: authentication. An attacker on the network can serve any certificate and the client will accept it. Worse, it normalises bypassing security checks on the team, making it harder to enforce proper certificate management later.

**Expired certificates.** This is the most common production incident. The fix is monitoring: alert when any certificate has fewer than 30 days of residual validity. cert-manager handles automatic renewal for certificates it manages. For the Kubernetes control plane PKI, use `kubeadm certs check-expiration` in a cron job or integrate it with your monitoring stack.

```bash
# Extract days remaining for any certificate
echo | openssl s_client -connect api.example.com:443 2>/dev/null \
  | openssl x509 -noout -enddate \
  | awk -F= '{print $2}' \
  | xargs -I{} date -d {} +%s \
  | xargs -I{} bash -c 'echo $(( ({} - $(date +%s)) / 86400 )) days remaining'
```

**SAN mismatch.** The hostname you are connecting to must appear in the certificate's SAN list. A certificate issued for `api.example.com` will not work for `10.96.0.1` (the API server's cluster IP) unless that IP is included in the SAN as an `IP Address` entry, a DNS name entry alone is not sufficient. This is a common issue when regenerating Kubernetes API server certificates after changing a cluster's IP ranges. Note: TLS clients strip the port before performing SAN matching, so the port number (`6443`) is irrelevant to hostname verification; only the IP or DNS name matters.

```bash
# Check what SANs are on the API server certificate
openssl s_client -connect <control-plane-ip>:6443 2>/dev/null \
  | openssl x509 -noout -text | grep -A5 "Subject Alternative Name"
```

**`unknown authority` errors.** This error means the certificate's issuer is not in the client's trust store. For internal CAs, you need to distribute the root CA certificate to all clients. In Kubernetes, the cluster CA is distributed automatically via the `kube-root-ca.crt` ConfigMap that exists in every namespace. Applications that need to trust the cluster CA can mount it:

```bash
kubectl get configmap kube-root-ca.crt -n default -o jsonpath='{.data.ca\.crt}' \
  | openssl x509 -text -noout | grep -E "Subject:|Not After"
```

**Wildcard certificate abuse.** A wildcard certificate for `*.example.com` covers every subdomain at one level. If the private key is ever exfiltrated, every subdomain is compromised until the certificate is revoked and reissued. For high-value domains, prefer per-service certificates, cert-manager makes this inexpensive.

> **Note on revocation.** Beyond expiry, certificates can also be revoked before their natural end-of-life. This is handled through two mechanisms: **CRL (Certificate Revocation List)** (a periodically published list of revoked serial numbers signed by the CA) and **OCSP (Online Certificate Status Protocol)** (a real-time query/response protocol where clients ask the CA "is this cert still valid?"). **OCSP stapling** is the modern best practice: the server fetches and caches the OCSP response itself and sends it to clients during the TLS handshake, avoiding a round-trip to the CA and improving both performance and privacy. If you are operating an internal CA, check that your CRL distribution points and OCSP endpoints are reachable from all clients, or incidents from revoked certificates will go undetected.

---

## Conclusion

TLS is not magic and it is not just a padlock icon. It is asymmetric cryptography, a global hierarchy of certificate authorities, a precise handshake protocol, and a naming system that binds public keys to identities. Every piece of modern infrastructure (from the Kubernetes API server to the connection between your pod's sidecar and the mesh control plane) is built on these same structures.

Once you have read a raw certificate with `openssl x509 -text -noout`, traced a chain of trust manually, and looked inside `/etc/kubernetes/pki/`, the error messages stop being cryptic. You know which link in the chain is broken, you know where to look, and you are not waiting for someone else to explain it.

If the views of this article will be high like the previous 2 of this series, I'll produce a new episode in the future.

---

## Further Reading

- [RFC 5280: Internet X.509 Public Key Infrastructure](https://datatracker.ietf.org/doc/html/rfc5280)
- [The Illustrated TLS 1.3 Connection](https://tls13.xargs.org/)
- [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/online/)
- [Kubernetes PKI and certificates](https://kubernetes.io/docs/setup/best-practices/certificates/)
- [cert-manager documentation](https://cert-manager.io/docs/)
- [SPIFFE and SVID specification](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/)
