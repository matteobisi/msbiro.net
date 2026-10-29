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
description: "Silos return dressed as autonomy. A cloud native security leader on the episode that taught him the difference between ownership and isolation, and what to fix before the next deadline."
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

A regulation lands with a date, and a team decides to solve its part directly.

That sentence is the article in miniature. Here is the episode. I lead a cloud native security team, and a department in a regulated programme had a NIS2 deadline, capable engineers and a strong sense that asking for help would cost them a sprint. They picked a runtime security tool, deployed it themselves and moved on.

Two months later, the deadline was missed. The tool was not the failure. Nobody had established the security policy baseline that tells a runtime tool what normal looks like and what must be blocked. The team that could have written it was never called.

When we spoke afterwards, nobody was defensive. The answer was "we had it under control". In a narrow sense, they did. They had a tool, a dashboard and a plan. What they did not have was the definition that makes the tool mean anything, and a reason to believe we would answer quickly if they asked.

In the last two years I have heard "we can handle this ourselves" more often than "that is not my job". The old silo was easy to spot because it was absent. The new one shows up, ships something and calls that ownership.

I am not writing this to argue that teams should ask security for permission. I am writing it because silos that look like autonomy survive longer, and because security teams often make going alone the rational choice.

## The new version of the silo

The old silo was a wall. The new one is a door. The team walks through it, closes it behind them and reports progress. Both leave a gap in the conversation, but only the new one looks like progress.

| Signal | Old silo | New silo |
| --- | --- | --- |
| Language | "That is not my job" | "We can handle this ourselves" |
| Presence | Nobody shows up | The team shows up, does not ask |
| Failure mode | Work never starts | Work starts, then stalls at the missing piece |
| Cost | Late, visibly | Late, after looking like progress |

The label does not save you. You can call a team cloud native security, platform security or DevSecOps, and the silo survives all three names. Renaming a team is not an operating model.

Three things make the new silo likely.

The first is a deadline. NIS2 and DORA turn a general sense that security matters into a date in a shared calendar. A date creates urgency, and urgency rewards the fastest visible action. Buying and deploying a tool is visible. Writing the policy baseline, deciding what to block, defining the exception path and agreeing who verifies it are slower work and less photogenic. So the tool ships first and the definition of done arrives late.

The second is that security can look like an approval queue. If every question to my team creates a review, a list of objections and a two-week wait, then a well-run engineering team is behaving rationally when it routes around us. I have written before that security becomes a bottleneck when it sits inside routine implementation decisions. The consequence is predictable: teams learn that talking to security is how work dies. Autonomy then becomes the polite name for avoidance.

The third is that we reward the wrong moment. A team that ships something alone gets a demo, a dashboard and a mention in the programme update. A team that asks early gets no visible credit, because the value shows up later as a problem that did not happen. If the organisation only measures what shipped, it will keep producing teams that skip the conversation.

The failure that follows is not the tool failing. It is the tool succeeding at something nobody defined.

Runtime detection without a policy baseline produces alerts with no notion of normal. Admission control without an exception process becomes a gate teams disable on a Friday. A detection pipeline without a log retention decision produces evidence nobody can use. In the episode, the platform looked more secure and could not answer the simplest question an auditor asks: what did you decide was unacceptable, and where is that written down?

A baseline is not a policy document. It is a short, testable answer to a small set of questions:

- What workloads are allowed to run, and with which security context?
- Which registries and image provenance are accepted? I have argued for a [hardened container image catalog](/posts/hardened-images-catalog-2026-non-negotiable/) before.
- What must be blocked at admission, and what can only be alerted?
- Who can approve an exception, and when does it expire?
- Which logs are retained, for how long, and who reads them? The [audit log work for SOC evidence](/posts/kubernetes-audit-logs-soc-evidence/) is the same split in practice.

Then the self-inflicted part, which is the one security leaders dislike hearing. If my team makes a baseline hard to find, a decision slow to get and an exception expensive to approve, we are not protecting autonomy. We are manufacturing the conditions for a shadow programme.

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

In the episode, the baseline did not exist yet, but the team that knew how to write it did. One conversation at kick-off would have been enough. The rest of the fix is keeping that conversation cheap: a baseline teams can find, a contact who answers in the same sprint, and an exception with an owner and an expiry date.

If a dependency has no owner or no date, the team is not autonomous yet. It is alone. I check those three things with the engineering manager at kick-off and again when the first sprint slips.

If the deadline is still missed, I try not to start with who ignored the process. I ask what made going alone the rational choice. Was the baseline findable, and was the contact responsive? Was there capacity to do it properly, and did anyone with authority decide the trade? That separates a team that made a mistake from a system that keeps making the mistake for them.

The old silo said "that is not my job" and left the room. The new one says "we have got this" and closes the door. Both leave an outcome nobody can fully see. Autonomy is real when a team can act and still knows where the rest of the answer lives; a silo is what remains when that link costs more than it should.

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
