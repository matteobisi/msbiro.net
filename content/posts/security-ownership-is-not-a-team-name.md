---
title: "Security Ownership Is Not a Team Name"
date: 2026-10-27T08:00:00Z
tags: [
  "security-leadership",
  "leadership",
  "engineering-leadership",
  "devsecops",
  "cybersecurity",
  "governance",
  "risk-management",
  "accountability",
  "silos"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Security ownership fails when a team holds the ticket but not the authority, or tries to solve it alone. A security leader on decision rights and silos."
canonicalURL: "https://www.msbiro.net/posts/security-ownership-is-not-a-team-name/"
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
    alt: "Security ownership: authority, capacity and decision rights in engineering organisations"
    caption: "Security ownership needs authority, not just a name"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

A security finding stalls, and the ticket looks perfect. It has a team name, a severity, a due date and a link to the policy it violates.

Three weeks later it is still open. Security says engineering has not implemented the control. Engineering says the requirement is unclear, the sprint is full, or the fix would break a release. The engineering manager asks whether an exception is acceptable. Security asks who owns the risk. Nobody is wrong, and nothing moves.

I no longer read this as a lack of awareness. Nobody in that thread argues that security does not matter. The organisation wrote a name in the owner field and assumed that was security ownership. It is not.

**Security ownership is not a team name. It is the ability to make a decision, act on it, and answer for the outcome.**

I write this as someone who leads in security, so I will say where I think the problem sits. It is not only that engineering deprioritises security. Security leaders are part of the mechanism, and so is the opposite reflex, teams deciding they can handle security alone.

## A name in the owner field is not ownership

Responsibility, accountability and ownership get used as synonyms. They separate the moment priorities collide. The person responsible does the work, the person accountable answers for the result, and neither can do much if nobody has the authority to change the plan, the budget or the release date. When a ticket stalls, I ask what it assumed and what was actually missing.

| The ticket assumes | What is usually missing |
| --- | --- |
| A vulnerability has a technical owner | A funded remediation window, so the fix competes with features and loses |
| The standard requires multifactor authentication | Anyone who owns the legacy integration that cannot support it |
| An incident runbook exists | Someone with the authority to suspend the service |
| An exception was approved | An expiry date and a person who has to remove it |

The runbook row is the one I use when I want people to feel the problem. At 02:00 during an incident, who can decide to take a revenue generating service offline? If the answer is "we would need to get the right people in a meeting", the organisation has no decision process. It has a dependency on negotiation, and the cost is paid during the incident.

The frameworks say the same once you read past the headline. NIST CSF 2.0 outcome GV.RR-02 asks for roles, responsibilities and *authorities* to be established, communicated, understood and *enforced*. Authorities is the word most ownership models skip, and enforced is the one most cannot demonstrate. [NIS2 Article 20](https://eur-lex.europa.eu/eli/dir/2022/2555/oj) and [DORA Article 5](https://eur-lex.europa.eu/eli/reg/2022/2554/oj) put accountability on the management body, not on a security team. I covered what that means for a platform team in [Who Owns NIS2 and DORA in a Platform Team](/posts/who-owns-nis2-dora-in-a-platform-team/).

## Security teams are part of the problem

Security teams are rarely forced into ownership they cannot deliver. More often we walk into it. Taking the ticket is the path of least resistance: it keeps us in the room, it feels like control, and it avoids the harder conversation about who funds the fix. Over time it becomes the operating model, and two things follow.

The first is the bottleneck. A small group of specialists ends up inside routine implementation decisions that belong to the teams operating the systems. Security becomes an approval queue, and teams learn to work around it.

The second is accountability without control. Security is judged on whether risk goes down, yet it controls none of the backlog, staffing, architecture or release dates that decide the answer. I consider this the more dangerous pattern, because from the outside it looks like leadership. A security leader accountable for outcomes they cannot influence is not owning risk. They are absorbing it on behalf of the people who could reduce it.

None of this is an argument for security to step back. Security should define what "good enough" means in terms engineers can test, describe the risk in business language, challenge exceptions and delays, and verify independently that controls stay effective. It should not merge every policy change, chase every team for remediation, or act as the default risk acceptor because it understands the technical detail.

There is a harder position behind that. **Decline to be the named owner of a risk you can neither fund nor fix, and say so in writing.** That is not obstruction. It pushes the question to the person who can answer it, and it replaces a comfortable fiction on a dashboard with a decision somebody has to make.

## Silos come back dressed as autonomy

We have talked about breaking silos for years, and I still see them return with a better excuse. The old silo said "that is not my job". The new one says "we can handle this ourselves", and it is harder to challenge because it sounds like the ownership we spent years asking for.

Here is the version I recognise. A regulation lands with a deadline, and a department decides to solve its part directly. It is capable and motivated, and it sees asking for help as slowing down. It picks a runtime security tool and deploys it. Two months later the deadline is missed, and not because the tool failed. Nobody had established the security policy baseline that tells the tool what normal looks like and what must be blocked. The DevSecOps team that could have defined it was never called, because everyone believed they had it under control. I describe this as a pattern, not a single incident: a team owned the outcome and skipped the people who held the missing piece.

The new silo is partly a consequence of the previous chapter. If security is an approval queue, going alone is a rational choice, and if asking the DevSecOps team means weeks of waiting and a list of objections, teams will learn to avoid it. Autonomy is not the problem. Autonomy without a cheap way to pull in the right people is. So I treat ownership as including the obligation to know what you do not know. In practice that means a published baseline teams can find before they choose a tool, a named contact who gives a real answer in the same sprint, and one question at kick-off for anything driven by a regulation or customer commitment: who else holds a piece of this, and have we spoken to them? If teams go around us, the first thing I check is whether we made asking expensive.

## Decisions, exceptions and capacity

"Who owns this control?" is too blunt a question, so I replace it with four decisions. Each can have a different holder, and each fails in its own way.

| Decision | Who should hold it | How it usually fails |
| --- | --- | --- |
| Define the expected outcome | Security, risk or architecture | A policy statement nobody can test |
| Implement and operate it | The team that owns the system | Security ends up doing it, or it is done once and never maintained |
| Accept the residual risk | A role with authority over the service and budget to remediate | Whoever understands the technical detail, or security by default |
| Verify it stays effective | Security or another independent function | Implementation is treated as proof, and drift goes unnoticed |

Risk acceptance deserves most of the attention, because it is the decision most often handed to the wrong person. **The person who accepts a risk must be able to fund its remediation, or carry the consequence if it never happens.** Someone who can sign but cannot fund is not accepting a risk. They are witnessing it. That is why the manager who requests an exception, not the security reviewer, has to carry it, a point I made in the NIS2 and DORA article.

Exceptions are where this gets tested. A legacy dependency or a real technical limitation can make immediate compliance impractical, and a mature organisation will have exceptions; the goal is to make them deliberate, visible and temporary. NIST CSF 2.0 outcome ID.RA-07 asks for exceptions to be managed, assessed for risk impact, recorded and tracked. The two details that disappear first are the remediation owner and the expiry date. The release ships, and the temporary arrangement becomes permanent because nobody was assigned the work of removing it. When the same exception renews month after month, I do not look for a careless engineer. I look at the roadmap. Either the fix was never funded, the platform has no supported path, or the business has chosen to accept the risk without saying so.

That is a capacity question. A team cannot own an outcome it has no realistic capacity to deliver, yet security work is routinely added to a roadmap as if it were free. CSF 2.0 outcome GV.RR-03 expects resources to match the risk strategy, roles and policies. When everything is urgent, work without a product deadline loses, and then security escalates while engineering negotiates exceptions. Both are reacting to the same planning failure. An engineering leader has to make the trade explicit: if this control is required, what moves, and if nothing can move, who accepts the risk, for how long and on what evidence? A backlog is a statement of priorities whether or not leadership intends it. If security work keeps losing to feature work, the organisation is making a choice, and the only question is whether the accountable people know. This is one more reason I keep arguing that [engineering managers are your real culture](/posts/engineering-managers-culture-cto-force-multiplier/): they hold the roadmap where that choice is made.

Counting assigned tickets tells you little. I watch the age of high-risk findings, the age and renewal rate of exceptions, the time it takes to get a risk decision, and whether remediation work is actually funded on the roadmap. These are diagnostic signals, not targets, and I review them with engineering and product leadership in the room, because a review that only security attends turns into another escalation queue.

## The question behind the ticket

Stronger ownership can turn into a hunt for someone to blame, and that is not what I am arguing for. After a control fails I ask what happened and what made it possible: was the supported path usable, was the risk decided by the right person, was there capacity to fix it. That separates an isolated mistake from a system that keeps producing the same failure. It is the principle behind [From Delegation to Ownership](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/): you cannot ask people to own outcomes and then punish them for surfacing problems.

So when a finding stalls, I try not to start with who owns the ticket. I ask who can decide what happens next, whether they have the authority and the information, which team has to implement it, whether that team has the capacity, and who else holds a piece of the answer. If those questions have no clear answers, adding another team name will not help.

The job of a security leader is not to own every security task. It is to build an organisation where the right people can decide, the teams responsible can act, nobody solves it alone by default, and unresolved risk reaches someone empowered to do something about it.

If everyone is responsible for security but nobody can decide, that is not shared ownership. It is a queue with a security label.

## References

- Matteo Bisi, [Who Owns NIS2 and DORA in a Platform Team](/posts/who-owns-nis2-dora-in-a-platform-team/), the companion article on turning regulatory requirements into engineering work.
- Matteo Bisi, [Engineering Managers Are Your Real Culture](/posts/engineering-managers-culture-cto-force-multiplier/).
- Matteo Bisi, [From Delegation to Ownership](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/).
- [NIST Cybersecurity Framework 2.0 (CSWP 29)](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf), Govern function: GV.RR-02 and GV.RR-03; Identify function: ID.RA-07.
- [Directive (EU) 2022/2555 (NIS2), Article 20](https://eur-lex.europa.eu/eli/dir/2022/2555/oj)
- [Regulation (EU) 2022/2554 (DORA), Article 5](https://eur-lex.europa.eu/eli/reg/2022/2554/oj)

*This post describes an engineering operating model, not a legal interpretation or a universal assignment of responsibilities. Adapt it to your organisation's structure and risk management process.*
