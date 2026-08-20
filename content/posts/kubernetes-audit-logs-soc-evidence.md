---
title: "Kubernetes Audit Logs: Designing the Evidence Your SOC Actually Needs"
date: 2026-08-21T09:00:00Z
tags: [
  "kubernetes", "audit-logging", "soc", "siem", "devsecops",
  "cloud-native", "cybersecurity", "cis", "cnapp", "sentinelone",
  "compliance", "incident-response"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "How Kubernetes audit logging works, what the defaults leave out, and how to tune the audit policy to collect the evidence a SOC needs without drowning analysts in noise. Includes the CIS Kubernetes Benchmark recommendations and how to ship audit logs to your SIEM."
canonicalURL: "https://www.msbiro.net/posts/kubernetes-audit-logs-soc-evidence/"
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
    alt: "Kubernetes audit logs flowing from the API server to a SIEM for SOC analysts"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

One of the most important topics my team has been following this year is designing and implementing a SOC service for cloud-native workloads at our company. We started with the preventative layer. We designed and tuned a solid set of posture rules enforced by SentinelOne Singularity, covering misconfigurations, risky privileges, and known bad patterns across our clusters. That part is working well.

The phase we are in now is the second half of the same problem. Posture tells you what is vulnerable or misconfigured. It does not tell you what happened. For that you need evidence, the raw record of who did what and when, so that a detection has something to attach to and an analyst has something to investigate. We are collecting two streams of evidence into our internal SIEM: the ingress controller logs from HAProxy, and the Kubernetes audit logs from our clusters. The goal is to make both available to our SOC analysts as evidence when SentinelOne raises a detection, as an additional resource when the SIEM itself generates an alert, and as an automated correlation source alongside CNAPP findings.

This article is about the Kubernetes audit log half of that work. What the audit log is, how it works, what you get by default, and how to tune it so that you collect the evidence that matters while keeping a sane balance between security and the noise that makes a SIEM unusable.

---

## Why Audit Logs Are the Missing Half

Posture tools like a CNAPP answer "what could go wrong." They look at a cluster and report that a pod is running as root, or a role has more permissions than it needs. That is a snapshot of risk, and it is enormously valuable, but it is not evidence of an incident.

Audit logs answer "what actually happened." Every request to the Kubernetes API server, every `kubectl apply`, every service account token used to read a secret, every attempt to create a privileged pod, is recorded as an audit event. When a detection fires, the audit log is where you go to reconstruct the sequence that led to it.

The two layers are complementary. SentinelOne tells the analyst that something suspicious happened on a node. The audit log tells the analyst who, what, when, and from where, using which credentials, against which resource. Without the second half, a detection is a headline with no article. Without the first half, the audit log is a firehose with no reason to read it.

---

## How Kubernetes Audit Logging Works

Kubernetes audit logging is a feature of the kube-apiserver. It does not require an agent on every node. It does not need a sidecar. It is a first-class part of the control plane, and it works by recording a chronological set of events for every request that reaches the API server.

Each event is built from a handful of dimensions, and understanding them is the whole game:

| Dimension | What it captures |
| --- | --- |
| Stage | Where in the request lifecycle the event was recorded |
| Level | How much detail the event contains |
| User | Who made the request (username, groups, UID) |
| Source | Where the request came from (source IP, user agent) |
| Object | What the request touched (resource, namespace, name) |
| Verb | What action was attempted (get, list, create, update, delete) |
| Result | Whether it was allowed or denied |

The **stages** are the four points in the request lifecycle:

- `RequestReceived`: the request arrived at the API server.
- `ResponseStarted`: response headers were sent, before the body completed.
- `ResponseComplete`: the full response was sent.
- `Panic`: an unexpected error occurred.

Most implementations record at `ResponseComplete`, which captures the final result of the request. Recording at `RequestReceived` as well doubles the volume for little investigative value, which is why you can omit it.

The **levels** are the real tuning knob, and they map to a clear trade-off:

| Level | What you get | Cost |
| --- | --- | --- |
| `None` | Nothing logged for matching requests | No evidence |
| `Metadata` | Who, what, when, source, and result. No request or response body. | Low |
| `Request` | Metadata plus the request body. Not the response. | Medium |
| `RequestResponse` | Everything, including the response body | High, and risky |

The critical insight is that `RequestResponse` is where secrets leak. If you log the response body of a `get` on a Secret, the plaintext secret value lands in your log, which then lives in your SIEM, which is now a bigger and softer target than the cluster ever was. Most policies should never use `RequestResponse` for Secrets, and should use `Metadata` as the default.

---

## The Defaults: What You Get Out of the Box

Here is the part that surprises people. Kubernetes does not enable audit logging by default.

A stock cluster, whether it is kubeadm, a kube-apiserver you built yourself, or many managed offerings in their default configuration, records no audit log at all. The kube-apiserver only starts logging when you point it at an audit policy and a destination, neither of which is configured automatically.

The destination can be either of two backends:

- **Log backend**: the kube-apiserver writes audit events to a file on the control plane node (`--audit-log-path`).
- **Webhook backend**: the kube-apiserver sends audit events as JSON batches to a remote endpoint you control (`--audit-webhook-config-file`), which is the natural integration point for a SIEM.

Managed clusters handle this differently. EKS sends audit logs to CloudWatch Logs when you enable it. GKE surfaces Kubernetes audit events in Cloud Audit Logs. AKS routes them through Azure Monitor diagnostic settings. Each has its own toggle, and in each case the toggle defaults to off or to a minimal subset. Before you tune a policy, the first job is to confirm the log is being generated at all.

The second default problem is volume. When audit logging is turned on with a naive policy that captures everything at `RequestResponse`, the API server can generate enormous amounts of data. Kubernetes is chatty. Kubelets, controllers, and operators continuously list and watch resources. A policy that records every `list` and `watch` at full detail will bury your SIEM and, worse, will bury the one interesting event in a mountain of routine polling.

---

## Designing the Audit Policy

The audit policy is a YAML file that tells the kube-apiserver what to log. It is evaluated in order, and the first matching rule wins, which means you build it from the most specific rules down to a catch-all default.

A minimal, useful policy looks like this:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Secrets and sensitive resources: metadata only, never the body.
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]

  # High-value, low-volume actions: full request and response.
  - level: RequestResponse
    verbs: ["create", "update", "patch", "delete"]
    resources:
      - group: ""
        resources: ["pods", "serviceaccounts", "clusterroles", "rolebindings"]
      - group: "rbac.authorization.k8s.io"
        resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]

  # Authentication and authorization decisions: request only.
  - level: Request
    resources:
      - group: ""
        resources: ["tokenreviews", "subjectaccessreviews"]

  # Everything else: metadata, which is enough for reconstruction.
  - level: Metadata
```

The thinking behind each tier is simple. Secrets get `Metadata` because the body is the thing you must never store. Mutating actions on security-relevant resources get `RequestResponse` because those are exactly the actions a SOC analyst needs to inspect after an incident. The default catch-all is `Metadata`, because who/what/when/result is enough to reconstruct most investigations without the cost of full bodies.

You can scope rules by user, group, namespace, verb, resource, or non-resource URL. You can add an `omitStages` field to drop `RequestReceived` and halve your volume. You can even exclude the kubelet and controller noise explicitly, though in practice the default `Metadata` catch-all is what keeps the routine traffic from bloating the log.

---

## The Balance: Security vs. Noise

The entire art of audit policy design is the balance between evidence and noise, and there are a few rules of thumb that keep you sane.

Log the decisions that matter. The single most valuable thing an audit log records is the authorization result. Every `create`, `update`, `delete`, and every access to a sensitive resource, carries an annotation showing whether it was allowed or denied, and by whom. Denied requests are often more interesting than allowed ones, because they are where reconnaissance and misconfiguration show up first.

Be deliberate about request bodies. Full request bodies are invaluable when investigating a `create pod` that deployed something malicious, and a liability when investigating a `get secret`. Scope body logging to the mutating verbs on security-relevant resources, and keep everything else at metadata.

Exclude the noise sources you already trust. Kubelet heartbeats, controller reconcile loops, and operator watches are constant and predictable. If you find a specific service account or user generating the majority of your events with routine `get` and `list` traffic, exclude or downgrade them explicitly. Your SIEM budget should go to the unexpected, not the routine.

Test against a real incident. The only honest way to know if your policy captures the evidence you need is to rehearse a detection and then ask whether the audit log answered the questions an analyst would ask. If you cannot reconstruct the sequence from the events you collected, the policy is too sparse. If the analyst cannot find the interesting event in the volume, it is too noisy.

---

## Industry Standards: The CIS Kubernetes Benchmark

When you tune audit logging, there is a widely accepted baseline to compare against. The CIS Kubernetes Benchmark, which kube-bench and the CNAPP vendors implement, has a set of recommendations for audit logging on the kube-apiserver. The exact section numbers shift between benchmark versions, but the substance has been stable.

The scored recommendations cover:

- Enable audit logging by setting `--audit-log-path` to a valid file.
- Set `--audit-log-maxage` to retain logs for an appropriate period, commonly 30 days or more.
- Set `--audit-log-maxbackup` to a sensible retention count, commonly 10.
- Set `--audit-log-maxsize` to bound individual files, commonly 100 MB, before rotation.
- Ensure the audit policy is defined and that it does not log full request and response bodies for sensitive resources like Secrets.

The retention recommendations matter because an incident is often discovered days or weeks after the fact. If the log has rotated away, the evidence has rotated away with it. A 30 day window is a reasonable floor, and you should reconcile it against your SIEM's own retention and your organization's compliance obligations.

The final point is the one most worth internalizing. CIS does not merely say "turn on audit logging." It says turn it on, keep enough of it, and do not record secrets. Those three ideas are the whole discipline in miniature.

---

## Shipping to the SIEM

Collecting the log is only half of the work. Making it useful to a SOC requires getting it into the SIEM in a form analysts can query and correlate.

The webhook backend is the standard path. The kube-apiserver can batch audit events and POST them to a webhook sink, typically a log shipper or a dedicated collector that normalizes and forwards them. This avoids scraping files off control plane nodes and gives you a steady stream of structured JSON.

The fields you want normalized and indexed are the ones an analyst filters on repeatedly:

- The `user` and `user.groups`, to answer "who."
- The `verb` and `objectRef.resource`, to answer "what."
- The `sourceIPs` and `userAgent`, to answer "from where."
- The `responseStatus.code` and the authorization annotation, to answer "was it allowed."
- The `requestReceivedTimestamp`, to place the event in a timeline.

The real value appears when these correlate with the rest of your stack. A SentinelOne detection on a node can be enriched with the audit events for the pod that ran on it, which shows who created the pod and with what privileges. An SIEM alert on anomalous network traffic can be enriched with the ingress logs you are already collecting, which shows the request, and the audit logs, which show the deployment behind it. That chain, detection to network to deployment to identity, is exactly the story a SOC analyst needs to go from "something happened" to "here is what happened and who is responsible."

---

## What We Are Doing With It

For our SOC service, the audit log is one leg of a three-part evidence chain. The posture rules in SentinelOne give us the standing risk picture. The HAProxy ingress logs give us the external-facing request evidence. The Kubernetes audit logs give us the internal control-plane evidence, the identity and authorization story that ties the other two together.

We ship audit events to the SIEM, normalize the fields an analyst actually queries, and correlate them with SentinelOne detections and CNAPP findings. The posture layer tells us where to look. The audit layer tells us what happened when someone actually touched it.

The lesson of the project so far is that audit logging is not a checkbox. It is a design exercise. The defaults leave it off, the naive configuration drowns you, and the correct configuration is the specific middle ground where you capture the decisions that matter without storing the secrets that would make your log a liability. That balance is the entire job.

---

## References

- [Kubernetes Audit Logging documentation](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
- [Kubernetes Audit Policy reference](https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/)
- [Kubernetes kube-apiserver reference (audit flags)](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
- [Aqua Security kube-bench](https://github.com/aquasecurity/kube-bench)
- [Amazon EKS audit log documentation](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-logs.html)
- [Google Kubernetes Engine audit logging](https://cloud.google.com/kubernetes-engine/docs/concepts/audit-logging)
- [Azure Kubernetes Service monitoring](https://learn.microsoft.com/en-us/azure/aks/monitor-aks)
