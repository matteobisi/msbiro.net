---
title: "Kubernetes Audit Logs and HAProxy Logs for SOC Evidence"
date: 2026-08-2T13:00:00Z
tags: [
  "kubernetes", "audit-logging", "haproxy", "ingress-controller",
  "soc", "siem", "devsecops", "cloud-native", "cnapp",
  "runtime-security", "incident-response"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how Kubernetes audit logs and HAProxy logs complement CNAPP posture and runtime security to give SOC analysts actionable evidence."
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
    alt: "Kubernetes audit logs and HAProxy ingress logs flowing to a SIEM for SOC analysts"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

One of the most important topics my team has been following this year is designing and implementing a SOC service for cloud-native workloads at [ReeVo](https://www.reevo.it/en/). We started with the preventative layer. We designed and tuned a solid set of posture rules through our CNAPP solution, covering misconfigurations, risky privileges, and known bad patterns across our clusters. The platform also gives us runtime-security capabilities to identify suspicious behaviour as it occurs. That part is working well.

The phase we are in now is the second half of the same problem. Posture tells you what is vulnerable or misconfigured. It does not tell you what happened. For that you need evidence, the raw record of who did what and when, so that a detection has something to attach to and an analyst has something to investigate. We are collecting two streams of evidence into our internal SIEM: the ingress controller logs from HAProxy and the Kubernetes audit logs from our clusters. They support CNAPP detections, SIEM alerts, and correlation with posture findings.

This article is about the evidence strategy behind both streams. We are tuning Kubernetes audit logs for control-plane activity, and defining the advanced HAProxy log fields that give analysts a useful view of internet-facing traffic. The objective is not to collect everything. It is to collect the information that turns a signal into an investigation and, when necessary, an early alert.

---

## Start With the Service, Not the Alert

It is easy to start a cloud-native security programme by enabling every available integration and turning every interesting signal into an alert. It feels like progress because the dashboard starts to fill up. But an alert alone is not a service. It is only an invitation to investigate.

The important work happens before that alert reaches the SOC. We need to decide what question the alert should answer, what evidence an analyst will need to make a decision, who owns the next action, and how the result will improve the service over time. Taking time to make those decisions is not a delay in delivery. It is how we avoid creating a noisy service that produces activity without creating confidence.

This is the approach we are taking. Rather than treating each security capability as an isolated product, we are designing a service in which the capabilities reinforce one another. Posture management identifies the conditions that increase risk. Runtime security identifies behaviour that requires attention. Ingress and audit evidence provide the history and context that make those signals understandable. Together, they give the SOC a path from a finding or alert to an informed response.

## Great Signals Still Need Raw Evidence

Our CNAPP solution is a powerful part of this model. It provides evaluated findings and detections, helping us identify relevant risk and focus the SOC on the activity most likely to matter. That assessment is valuable because it reduces the effort required to find a meaningful signal in a complex environment.

A SOC analyst will sometimes need more than a pre-evaluated finding. During an investigation, they may need the raw evidence of what was happening in the environment: the request reaching an internet-facing service, the response it received, the workload that served it, and the Kubernetes action that changed the workload or its permissions. The evaluated signal tells the analyst where to look. The raw evidence helps them establish what happened and decide what to do next.

A useful detection must help an analyst answer four basic questions: what happened, what was affected, who or what initiated the action, and what should happen next. Runtime security and posture management identify suspicious behaviour and exposure. Ingress and audit evidence supply the traffic, identity, timing, and change history needed to answer the remaining questions.

## Four Capabilities, One Security Story

We see posture management, runtime security, ingress evidence, and audit evidence as different views of the same environment.

| Capability | The question it helps answer | Its role in the service |
| --- | --- | --- |
| Posture management | What could be exposed or misconfigured? | It helps us reduce known risk and prioritise attention. |
| Runtime security | Is something suspicious happening now? | It gives us timely signals that need investigation or response. |
| HAProxy ingress logs | What is happening at the internet-facing edge? | They show the requests, clients, responses, and traffic patterns seen by exposed services. |
| Kubernetes audit logs | What changed, who did it, and when? | They show the control-plane actions, identities, and authorisation context behind a workload. |

No capability replaces the others. Strong posture reduces the opportunities available to an attacker, but it cannot prove what happened during an incident. Runtime detections provide urgency, but can lack the surrounding context needed for confident triage. HAProxy and Kubernetes audit logs provide complementary evidence, but are most useful when they are collected with a clear investigative purpose.

The service becomes stronger when these views can be correlated. A runtime detection involving a workload can be examined alongside its audit history and the ingress requests it served. A posture finding can be prioritised differently when it is associated with unexpected traffic. An ingress event can be linked to the workload and the change that put it in place. The analyst receives a story instead of several disconnected notifications.

## The Evidence Streams We Are Tuning

Kubernetes audit logs give us the internal control-plane perspective. We tune them to preserve the security-relevant actions, identities, and authorisation decisions that explain how a workload or its access changed. This is particularly important when an investigation needs to establish who created, modified, or granted access to a resource.

HAProxy gives us the external-facing perspective. We are defining an advanced log schema that retains the security-relevant fields an analyst needs to understand traffic reaching an exposed service. This includes the client source, request method and path, host, response status, request timing, bytes transferred, backend or service selected, and the TLS and connection context where relevant.

Both streams must be designed deliberately. They should provide enough context to investigate suspicious activity while respecting data-handling boundaries and avoiding the unnecessary collection of sensitive content. The useful question is not whether a field can be logged, but whether it will help an analyst assess risk, reconstruct an incident, or correlate the request with another signal.

HAProxy logs are valuable not only after an alert. They can also create early signals when the traffic profile of an internet-exposed application changes in a meaningful way. A sustained rise in relevant HTTP errors, measured against the normal behaviour of a service, may indicate a failing dependency, an application under stress, a deployment problem, or an attempted attack. The associated raw requests and connection context give the SOC the evidence to distinguish an operational issue from suspicious activity.

The same principle applies to changes in client behaviour, unexpected request patterns, or unusual traffic directed at a sensitive endpoint. The goal is not to alert on every error or every rejected request. It is to create high-confidence alerts where a meaningful deviation is supported by the context an analyst needs to act.

## Design the Investigation Before the Collection

The right question is not "how much logging can we enable?" It is "what evidence will an analyst need when this type of detection occurs?"

Starting from the investigation changes the design conversation. We can identify the actions and identities that need to be visible, the information that must be preserved for correlation, and the data that should never be collected because it creates unnecessary risk. Good evidence lets an analyst move from an alert to a defensible conclusion without spending hours searching across unrelated systems.

This approach also gives us a practical way to manage noise. Security teams do not need more data for its own sake. Routine activity that adds no investigative value should not obscure the activity that does. The goal is a signal set that is deliberate, explainable, and sustainable for both the platform team and the SOC.

It is equally important to treat the service as something that will evolve. We should test the evidence available during realistic scenarios, learn where an investigation lacks context, and refine the collection and correlation accordingly. A detection that cannot be explained is not finished. It is feedback for improving the service.

## What We Are Building

For our SOC service, the CNAPP provides the posture and runtime views that help us identify where attention is needed. HAProxy ingress logs add the external-facing request perspective. Kubernetes audit logs add the control-plane actions, identities, and changes behind each workload. Together, they give us evaluated findings and the raw evidence needed to investigate them.

Our focus is therefore not simply to enable logging. It is to make the two evidence streams useful in the moments that matter. That means shaping the Kubernetes audit policy and HAProxy log schema around investigations, normalising the information the SOC needs to query, and correlating it with the runtime and posture signals already available.

The outcome we want is an exceptional service, not an exceptional number of alerts. A well-crafted alert, supported by the right evidence and connected to the broader security picture, gives an analyst the confidence to act. That is why this combined approach can be a game changer. It turns a security signal into a story that the SOC can understand, validate, and respond to.

---

## References

- [Kubernetes Audit Logging documentation](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
- [HAProxy logging documentation](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/observability/logging/)
