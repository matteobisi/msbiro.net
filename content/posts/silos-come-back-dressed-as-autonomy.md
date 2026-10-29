---
title: "Silos Come Back Dressed as Autonomy"
date: 2026-10-29T08:00:00Z
tags: [
  "security-leadership",
  "leadership",
  "engineering-leadership",
  "cloud-native-security",
  "devsecops",
  "cybersecurity",
  "governance",
  "risk-management",
  "silos",
  "autonomy",
  "NIS2",
  "DORA"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Silos can return dressed as autonomy. An engineering leadership perspective on the difference between ownership and isolation, and how trust and collaboration help teams deliver."
canonicalURL: "https://www.msbiro.net/posts/silos-come-back-dressed-as-autonomy/"
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
    alt: "Silos disguised as autonomy in cloud native security teams"
    caption: "Autonomy without a cheap way to ask is a silo"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

A deadline creates pressure, and a team decides to solve its part of the problem directly.

I once saw this play out in a security initiative. The team had capable engineers, a clear objective and a reasonable concern that asking for help might slow them down. They selected a runtime security tool and got it deployed, expecting that to move the work forward.

The missing piece was not the tool. It was the security policy baseline that defines what normal looks like and what should be blocked. That baseline needed input from another team, but the dependency was not surfaced early enough.

Looking back, the decision made sense from one perspective: the team had a task to deliver and wanted to keep momentum. They had a tool, a dashboard and a plan. What was missing was the shared definition that would make the tool effective, and confidence that asking for help would lead to a timely answer.

In engineering, "we can handle this ourselves" can sound like a healthy sense of ownership. The old silo was easy to spot because it was absent. The newer version shows up, makes progress and can unintentionally leave a gap in the conversation.

This is not an argument for teams to ask security for permission. It is a reminder that autonomy works best when teams can act independently and still reach the people who hold part of the answer. Security teams also have a responsibility to make that collaboration easy, responsive and useful.

## The new version of the silo

The old silo was a wall. The newer one can be a door that closes without anyone noticing: the team keeps moving and reports progress, but a key conversation never happens. Both create gaps in understanding, though only one may look like progress at first.

| Signal | Old silo | New silo |
| --- | --- | --- |
| Language | "That is not my job" | "We can handle this ourselves" |
| Presence | Nobody shows up | The team shows up, does not ask |
| Failure mode | Work never starts | Work starts, then stalls at the missing piece |
| Cost | Late, visibly | Late, after looking like progress |

The label alone does not solve it. You can call a team cloud native security, platform security or DevSecOps, but collaboration still depends on how people work together. Renaming a team is not an operating model.

Autonomy is not automatically a silo. A team that owns its domain and can make its own decisions is what good engineering looks like, and sometimes that autonomy works exactly as intended. The silo appears when autonomy turns into self-sufficiency: the team stops checking who else holds a piece of the answer, reinvents the wheel, and forgets to involve the team that already has it. That is where the time goes. Collaboration and trust between teams are what keep the difference visible.

Three things make the new silo likely.

The first is a deadline. Regulatory requirements such as NIS2 and DORA can make security outcomes time-sensitive. Urgency often rewards visible progress: a tool deployed, a dashboard available, a milestone marked complete. Agreeing on expected behaviour, exception handling and verification may be less visible, but it matters just as much. A useful question at the start is whether the team has defined success beyond implementation.

The second is that security can look like an approval queue. If every question to a security team creates a review, a list of objections and a long wait, a well-run engineering team may reasonably try to work around it. Security becomes a bottleneck when it sits inside routine implementation decisions. The lesson for security teams is just as important: make guidance easy to find, provide a clear contact and respond while the work is still moving. Otherwise, autonomy can become a polite name for avoidance.

The third is what we choose to recognise. Shipping independently is easy to demonstrate. An early conversation that prevents rework is harder to measure, because its value may be a problem that never occurs. Leaders can help by recognising both delivery and the collaboration that makes delivery sustainable.

The risk is not always a tool failing. Sometimes it is a tool working as configured without a shared understanding of what it is supposed to achieve.

The same pattern appears across security engineering. Runtime detection needs an agreed baseline for interpreting alerts. Admission controls need a workable exception process. Detection pipelines need retention and ownership decisions if their evidence is to remain useful. These are general design dependencies, not shortcomings of any particular team or product. A useful test is whether everyone can answer: what outcome are we trying to achieve, and how will we verify it?

A baseline is not a policy document. It is a short, testable answer to a small set of questions:

- What workloads are allowed to run, and with which security context?
- Which registries and image provenance are accepted? I have argued for a [hardened container image catalog](/posts/hardened-images-catalog-2026-non-negotiable/) before.
- What must be blocked at admission, and what can only be alerted?
- Who can approve an exception, and when does it expire?
- Which logs are retained, for how long, and who reads them? The [audit log work for SOC evidence](/posts/kubernetes-audit-logs-soc-evidence/) is the same split in practice.

Security leaders have a part to play, too. When guidance is difficult to find, decisions take too long or exceptions are cumbersome, teams have an incentive to work around the process. Making support accessible is one practical way to protect both autonomy and shared standards.

The [ownership I keep asking engineers for](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/) only works when the people asking for it also have a cheap way to pull in the people who hold the missing piece. That trade is made by the manager who owns the roadmap, which is why I keep saying [engineering managers are the real culture](/posts/engineering-managers-culture-cto-force-multiplier/).

Autonomy is not the problem. Expensive help is.

Once a team decides to act, four decisions have to be made explicit, and they rarely belong to the same person.

| Decision | Who should hold it | How it usually fails |
| --- | --- | --- |
| Define the expected outcome | Security, risk or architecture | A policy statement nobody can test |
| Implement and operate it | The team that owns the system | Security ends up doing it, or it is done once and never maintained |
| Accept the residual risk | A role with authority over the service and the budget to remediate | Whoever understands the technical detail, or security by default |
| Verify it stays effective | Security or another independent function | Implementation is treated as proof, and drift goes unnoticed |

The third row is where most programmes quietly break. The person who accepts a risk must be able to fund its remediation or carry the consequence if it never happens. Someone who can sign but cannot fund is not accepting a risk. They are witnessing it.

The frameworks describe the same division of responsibility. NIST CSF 2.0 outcome GV.RR-02 asks for roles, responsibilities and authorities to be established, communicated, understood and enforced. Authorities is the word most operating models skip, and enforced is the one most cannot demonstrate. GV.RR-03 expects resources to match the risk strategy, so a team cannot be accountable for an outcome the roadmap never funded. ID.RA-07 asks for exceptions to be assessed, recorded and tracked.

NIS2 Article 20 and DORA Article 5 put the accountability on the management body, not on a security team. I went through what that means for a platform team in [Who Owns NIS2 and DORA in a Platform Team](/posts/who-owns-nis2-dora-in-a-platform-team/).

Nothing in those requirements asks a team to go alone. They ask that authority, funding and the exception path be explicit. A silo is autonomy with the dependencies hidden.

## Takeaway

A silo is not a team that stopped asking permission. It is a team that stopped asking for help.

Avoiding the next silo is easier than this article makes it sound. Do not reinvent the wheel. Ask the team that already owns the answer.

Often, the expertise needed to define a missing requirement already exists somewhere in the organisation. An early conversation can reveal that dependency before it becomes rework. The practical fix is to make collaboration easy: discoverable guidance, a responsive contact and an exception process with clear ownership and an expiry date.

A simple habit at kick-off is to ask who owns each dependency, when an answer is needed and how the team can get help. Revisiting those questions as the work progresses makes autonomy more resilient.

When work stalls, it is more useful to ask what made going alone seem like the best option than to start by assigning blame. Was guidance easy to find? Could the right people respond in time? Was there capacity to do the work properly, and were trade-offs made by someone with the right authority? These questions help teams improve the way they work together.

The old silo said "that is not my job" and left the room. The newer one says "we have got this" and may forget to open the conversation. Both can leave gaps that no one intended. Autonomy works best when a team can act, knows where the rest of the answer lives, and can reach the people who have it without unnecessary friction. Trust and collaboration are not the opposite of ownership; they are what make it sustainable.

## References

- Matteo Bisi, [Who Owns NIS2 and DORA in a Platform Team](/posts/who-owns-nis2-dora-in-a-platform-team/).
- Matteo Bisi, [Engineering Managers Are Your Real Culture](/posts/engineering-managers-culture-cto-force-multiplier/).
- Matteo Bisi, [From Delegation to Ownership](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/).
- Matteo Bisi, [In 2026 I Am Still Asked Why You Need a Hardened Container Image Catalog](/posts/hardened-images-catalog-2026-non-negotiable/).
- Matteo Bisi, [Kubernetes Audit Logs and HAProxy Logs for SOC Evidence](/posts/kubernetes-audit-logs-soc-evidence/).
- [NIST Cybersecurity Framework 2.0 (CSWP 29)](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf), Govern function: GV.RR-02 and GV.RR-03; Identify function: ID.RA-07.
- [Directive (EU) 2022/2555 (NIS2), Article 20](https://eur-lex.europa.eu/eli/dir/2022/2555/oj)
- [Regulation (EU) 2022/2554 (DORA), Article 5](https://eur-lex.europa.eu/eli/reg/2022/2554/oj)

*This post describes an engineering operating model, not a legal interpretation or a universal assignment of responsibilities. Adapt it to your organisation's structure and risk management process.*
