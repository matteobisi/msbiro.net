---
title: "Who Owns NIS2 and DORA in a Platform Team"
date: 2026-09-10T05:00:00+01:00
tags: [
  "leadership",
  "engineering management",
  "devsecops",
  "compliance",
  "cybersecurity",
  "NIS2",
  "DORA",
  "eu-regulation",
  "kubernetes",
  "platform-engineering",
  "cloud-native",
  "RACI"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "NIS2 and DORA fail when only security owns them. A Kubernetes platform RACI, plus the artefacts engineering managers must put on the board."
canonicalURL: "https://www.msbiro.net/posts/who-owns-nis2-dora-in-a-platform-team/"
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
    alt: "NIS2 and DORA ownership on a Kubernetes platform team: engineering manager, security, and RACI"
    caption: "NIS2 and DORA ownership is an engineering management problem"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

I keep seeing the same meeting. Someone from legal or risk drops a NIS2 control, a DORA article, or a "can we evidence this" request into the room. The Kubernetes platform team looks at security. Security looks at the platform team. A ticket lands on the security board, because that feels like the responsible place. Three sprints later the cluster has not changed, the evidence still does not exist, and everyone is slightly angry at the security lead.

I lead a DevSecOps team of consultants. We advise customers in financial services, where DORA is already the daily language, and in other industries that now have NIS2 on the table. I do not own their cluster. I turn EU regulation into work their platform team can finish.

NIS2 and DORA do not fail in the legal memo. They fail when nobody in engineering owns the artefact. I wrote in June that [engineering managers are the real culture](/posts/engineering-managers-culture-cto-force-multiplier/). Compliance is the same mechanism with a deadline. If the customer's engineering manager does not carry it, it does not land in sprint goals, and it does not land in the cluster. Security can write the policy. Security cannot merge the Helm chart.

---

## When Security Owns NIS2 and DORA

The default in a lot of cloud-native organisations is simple. Anything with a regulation in the title becomes a security ticket. The CISO, or the one senior security engineer who still answers Slack, becomes the owner of NIS2, DORA, and whatever arrives next. Platform engineers stay on product work. Application teams stay on features. The security person becomes a queue.

Two failure modes follow, and they feed each other.

The first is bottleneck. Every image exception, every privileged `securityContext`, every "we need this log in the SIEM" request waits on the same two people. Delivery teams learn that talking to security is how work dies. They start routing around it: a break-glass kubeconfig, a sidecar that was never reviewed, a registry that is not the catalog. You do not get non-compliance in a slide. You get it as shadow IT with a Jira label that still says "security".

The second is learned helplessness on the platform side. If compliance is always someone else's backlog, platform engineers never own the why. They implement a Kyverno policy because they were told to, then disable it when it blocks a release, then wait for security to notice. That is the opposite of the [ownership I keep asking for](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/). You cannot ask people to own outcomes and then hide the regulatory outcome in another team's board.

I have sat on both sides of that table with customers. I have seen the security queue. I have seen the engineering manager whose sprint already looked full. Neither version is a personality flaw. It is an org design that nobody named.

---

## What NIS2 Article 20 and DORA Article 5 Actually Assign

Both texts are blunt about where accountability sits, and it is not "the security team".

[NIS2 Article 20](https://eur-lex.europa.eu/eli/dir/2022/2555/oj) puts cybersecurity risk-management measures on the management body of essential and important entities. They approve the measures. They oversee implementation. They can be held liable. They have to follow training.

[DORA Article 5](https://eur-lex.europa.eu/eli/reg/2022/2554/oj) does the same for financial entities. The management body defines, approves, oversees, and remains responsible for the ICT risk management framework. Roles and responsibilities for ICT-related functions have to be assigned clearly. The CISO is not a substitute for that.

That sounds like a board problem. In practice it becomes a middle-management problem by Friday afternoon. The board will not patch `containerd`. The board will not decide whether kube-apiserver audit logs are retained for 12 months or 18. The board will not sit in the sprint planning where "ship the new tenant" competes with "we still have no evidence trail". Those choices are made by engineering managers, or they are not made at all.

The vacuum under the management body is where most programmes die. Legal translates the article. Security writes a control statement. Nobody is accountable for the Kubernetes object that would make the statement true. The same vacuum shows up in CRA programmes. The [EU Cyber Resilience Act roadmap](/posts/eu-cyber-resilience-act-practical-roadmap/) has a harder clock. Someone in engineering still has to own the work, or the accountability at the top is theatre.

---

## Put Artefacts on the Board, Not Regulation Names

A regulation is a reason. It is not a ticket. "NIS2 Article 21" cannot be demoed. "kube-apiserver audit policy shipped, logs in the SIEM, retention documented" can. The [audit log work I wrote about for SOC evidence](/posts/kubernetes-audit-logs-soc-evidence/) is the split in real life. The SOC asked for evidence. The platform had to own collection. Security had to say what "enough" looks like. If that split is fuzzy, you get a SIEM with nobody looking at it, or a cluster that still has audit logging set to `None`.

The artefact does not come from somebody reading a regulation and inventing a Kubernetes task. It comes from a traceability chain. Confirm the entity and service are in scope. Turn the requirement into a control question. Decide what evidence is acceptable. Then name the technical change or recurring operation that produces it. Risk, legal, and security own the interpretation. The platform team owns the part that exists in the cluster, pipeline, or runbook.

The map below is not a claim that every row is an explicit command in NIS2 or DORA. These texts are outcome-based. The links show the provisions that make the control relevant. Implementation, retention, and evidence still come from the organisation's risk assessment, sector rules, and internal policy. DORA references apply when the financial entity and the ICT service are in its scope.

| Operational artefact | Control question it answers | NIS2 provenance | DORA provenance |
| --- | --- | --- | --- |
| Hardened image catalog and rebuild pipeline | Can we reduce known weaknesses in deployed software, and rebuild promptly when a material vulnerability is found? | [Article 21(2)(d) and (e)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), supply-chain security and vulnerability handling | [Articles 8 and 9](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), ICT asset identification and protection/prevention |
| Secrets manager as the only credential path | Can we protect credentials, limit access, and show that secrets are not spread through code and cluster configuration? | [Article 21(2)(i) and (j)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), access control, asset management, and multi-factor authentication | [Article 9](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), protection and prevention measures |
| Kubernetes audit logs and ingress logs in the SIEM | Can we detect suspicious activity, investigate an incident, and retain evidence for the required period? | [Article 21(2)(b)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), incident handling | [Articles 10 and 17](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), detection capabilities and incident management |
| Admission policy, including restricted PSS and a ban on unpinned images | Can we prevent unsafe workload configurations before they reach production, while recording approved exceptions? | [Article 21(2)(f) and (i)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), security policies, access control, and asset management | [Article 9](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), protection and prevention measures |
| Incident runbook and 24-hour fact pack | Can on-call staff establish what happened, its impact, and the facts needed for a timely escalation? | [Articles 21(2)(b) and 23](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), incident handling and incident reporting | [Articles 17 and 19](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), incident management and reporting |
| SBOM generated and retained in CI | Can we identify what software was shipped when a supplier, package, or vulnerability requires investigation? | [Article 21(2)(d) and (e)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj), supply-chain security and vulnerability handling | [Article 28](https://eur-lex.europa.eu/eli/reg/2022/2554/oj), ICT third-party risk management |

The ticket names the artefact, links the control objective, states the evidence, and has a person who can get it done. That is a concrete task. It is not a request that the engineer act as the lawyer who decided the law applies.

### A RACI for a Kubernetes platform

Once that traceability exists, the RACI answers a different question: who gets the artefact over the line and keeps it true? Here is the Kubernetes platform RACI I push for. Adjust the names. Do not adjust the idea that security is rarely Accountable for the artefact. A [secrets manager](/posts/secrets-manager-catalog-2026-non-negotiable/) and a [hardened image catalog](/posts/hardened-images-catalog-2026-non-negotiable/) are the same pattern: a service the platform team runs, not an appliance security sprinkles on later.

| Artefact | Responsible | Accountable | Consulted | Informed |
| --- | --- | --- | --- | --- |
| Hardened image catalog and rebuild pipeline | Platform engineers | Platform engineering manager | Security (policy, CVE exceptions) | App teams, risk |
| Secrets manager as the only credential path | Platform engineers | Platform engineering manager | Security, IAM | App engineering managers |
| Kubernetes audit logs and ingress logs into the SIEM | Platform / SRE | Platform engineering manager | Security / SOC (required evidence) | Risk, CISO |
| Admission policy (restricted PSS, deny `:latest`) | Platform engineers | Platform engineering manager | Security (exceptions) | App teams |
| 24-hour incident clock: detect, page, assemble facts | On-call engineer | EM who owns that service | Security, legal, communications | Management body, CSIRT |
| Authority notification (NIS2, DORA, CRA in scope) | Security or nominated control function | CISO or nominated role | Legal, EM of the affected system | Management body |
| SBOM in CI for what you actually ship | Engineers who build the artefact | EM of that pipeline | Security (format, retention) | Procurement, risk |
| Exception to run privileged or with `hostNetwork` | Requesting engineer | Their engineering manager | Security | Platform, risk |

RACI is a communication tool. If you publish the table and never look at it in planning, you have stationery. Two rows get confused more than the rest.

### Detection is not NIS2 or DORA notification

Incident detection and authority notification are different jobs. If you make the on-call engineer "own NIS2 reporting", you will get panic or silence.

The engineer owns facts: what broke, when, blast radius, what you already did. The nominated control function owns the notice to the authority. The engineering manager owns whether the runbook was rehearsed before 02:00.

### Exceptions belong to the requesting engineering manager

Security can advise that `CAP_SYS_ADMIN` is a bad idea. Only the requesting team's manager can decide that the product risk is worth the exception, write it down, and expire it.

If security is Accountable for exceptions, every exception becomes a negotiation with the person everyone is trying not to bother. The real decision maker stays invisible.

---

## Put the Artefact in the Next Sprint

The test I use in a customer workshop is simple. Is the artefact in the next sprint, with a named engineering manager, or is it still a control statement on a security slide? A useful diagnostic is who is holding work labelled security or compliance that should not be theirs. A Kyverno policy the platform should own is one answer. A 40-page control mapping that belongs with risk is another.

Capacity is the part senior leaders skip. A platform team staffed for product only will treat every control as overtime. They will get the catalog, or the new tenant environment. They will not get both from the same three people unless someone lies in the roadmap. I would rather tell a customer to cut scope than put "DORA" on a slide and leave their engineers to find the hours in the evening. The [culture you invest in at engineering manager level](/posts/engineering-managers-culture-cto-force-multiplier/) includes the right to say the plan does not fit.

Security still sets the bar: what "restricted" means, which CVEs are exceptions, what a 24-hour fact pack must contain, which logs the SOC will actually use. It reviews exceptions, runs or supports authority notification, and trains engineering managers until they can have the conversation without forwarding every email to the CISO.

Consulted has to mean "answer in the same sprint", not "join a working group". If security cannot staff that, the honest move is to shrink the control set. People walk toward security when talking to it makes the work clearer and smaller. They walk around it when it does the opposite.

"We cannot evidence access to kube-apiserver for the last incident" in week two is useful. The same sentence the week before an on-site inspection is a resignation in slow motion.

---

## Look at the Engineering Manager First

The chain is short. The management body is accountable in the text. The engineering manager is accountable in the cluster. Security is the specialist you consult so the work is right. The engineer is responsible for the object that ships.

If you only staff the management body and security, you get policies and liability with no Kubernetes. If you only staff the engineering manager and the engineer, you get a fast platform you cannot explain to a supervisor.

The next time a regulation lands in the room, try not to look at security first. Look at the engineering manager who owns the system it describes. Ask them what artefact they will demo, and which sprint it is in. If they cannot answer, you do not have a security gap. You have a management gap, and you already know those are the expensive ones.

---

## References

- [Directive (EU) 2022/2555 (NIS2), Article 20](https://eur-lex.europa.eu/eli/dir/2022/2555/oj)
- [Regulation (EU) 2022/2554 (DORA), Article 5](https://eur-lex.europa.eu/eli/reg/2022/2554/oj)
- [Matteo Bisi, Engineering Managers Are Your Real Culture](/posts/engineering-managers-culture-cto-force-multiplier/)
- [Matteo Bisi, From Delegation to Ownership](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/)
- [Matteo Bisi, The EU Cyber Resilience Act: A Practical Roadmap](/posts/eu-cyber-resilience-act-practical-roadmap/)
- [Matteo Bisi, Kubernetes Audit Logs and HAProxy Logs for SOC Evidence](/posts/kubernetes-audit-logs-soc-evidence/)
- [Matteo Bisi, In 2026 I Am Still Asked Why You Need a Centralized Secrets Manager](/posts/secrets-manager-catalog-2026-non-negotiable/)
- [Matteo Bisi, In 2026 I Am Still Asked Why You Need a Hardened Container Image Catalog](/posts/hardened-images-catalog-2026-non-negotiable/)
