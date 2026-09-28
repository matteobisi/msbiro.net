---
title: "Career Ladders That Engineers Do Not Game"
date: 2026-09-28T08:00:00+01:00
tags: [
  "leadership",
  "engineering management",
  "career growth",
  "leveling",
  "career ladder",
  "performance management",
  "team culture",
  "people management",
  "hiring"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A career ladder is a contract, not a list of adjectives. How to write engineering levels in scope and decision rights instead of years of experience, how to spot the two failure modes that make teams game them, and what a manager has to do to make the thing real."
canonicalURL: "https://www.msbiro.net/posts/career-ladders-leveling-engineers-do-not-game/"
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
    alt: "A career ladder document next to a whiteboard covered in engineering level definitions"
    caption: "Writing levels engineers trust"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

I have written twice on this blog about the human side of leading engineers, once about [delegation and ownership](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/) and once about [my own move from senior engineer to team leader](/posts/from-senior-system-engineer-to-team-leader-journey-leadership-principales/). Both pieces were about philosophy. Philosophy is cheap and it is easy to publish, which is exactly why it is not where a team actually needs help.

The thing teams ask me for most often is much more boring. Write the levels down. Tell me what senior means before I have to guess, and tell me what I have to do to get there. If a promotion is going to be judged on impact and influence, then write impact and influence down somewhere I can point at, not "senior plus five years".

Most ladders I have seen fail, and they fail in one of two predictable ways. Either they describe seniority using adjectives, which means everyone projects their own meaning onto the document; or they are precise but invisible, technically accurate and completely unowned by anybody who has to live with them. Both failures produce the same symptom on the ground: engineers who conclude the ladder is a mechanism for extracting work, and managers who end up adjudicating every promotion from memory and vibes.

Here is what I have learned about writing ones that hold up.

---

## A Ladder Is a Contract, Not a Title

Before the writing, the model. When I say ladder, I mean an explicit agreement about how progression works: what each level is responsible for, what changes when you move up, who decides, and how you can see your progress before you ask.

If that contract is missing, what you have instead is a set of titles that drift with market pressure. Someone joins and they are senior because you needed to hire them. Someone else is senior because they have been there since before you started. A third is senior because their manager liked them. Nobody is gaming anything, because there is nothing to game. The ladder was never a rule, it was a label.

The practical test for whether a ladder is real: could a competent engineer read it and predict, with reasonable accuracy, whether they are operating at the next level or not? If the honest answer is that they would need to ask you in person, the ladder is decorative, and the conversation becomes a negotiation every single time.

---

## Write Scope and Decision Rights, Not Adjectives

The most common format I see is a bulleted list of soft traits. "Senior engineer demonstrates strong communication skills, shows initiative, and mentors other engineers." Every one of those words is unfalsifiable. They cannot be scored, and they cannot be argued with, which is exactly the problem, because the engineer who disagrees with a promotion decision has no way to contest it.

I write levels along three axes instead, and I try to keep the language boring enough that a lawyer would not blink.

**Scope of impact.** Who is affected by this person's work. Junior work is local, it is the well being of their own service and the direct reliability of it. Senior work extends to systems and services they do not own, and to the on-call burden of people they do not know. Staff and above work is measured by something outside engineering entirely, usually a business capability, a cost, or a risk number.

**Ambiguity.** How much of the problem arrives defined. The entry level receives well-scoped problems with a clear definition of done. The senior level receives a problem and owns the framing, which means they are the one deciding what "done" even means. The staff level receives a problem that someone above is not sure can be solved at all, and has to tell that person so early and clearly.

**Decision rights.** Who gets to call it, and who lives with the consequences. This is the axis most ladders skip, and it is the one people actually notice in their day to day work. A senior engineer who cannot make a decision without a meeting is a senior engineer in name only. Writing down where decision rights start, in terms of blast radius, is what makes a level feel real to the person holding it.

The reason these three work better than adjectives is that they can be demonstrated in a review, and they can be shown. When I tell an engineer they are not at the next level yet, I can point at the specific axis and the specific quarter. "Your impact is still scoped to your own service" is a hard thing to hear and a fair thing to say. "You are not a senior engineer" is neither.

---

## The Two Tests That Catch Gaming

Ladders get gamed because the incentives reward the wrong behavior, not because engineers are adversarial. Two questions do most of the work of finding those spots.

**If everyone on the team were promoted to the next level tomorrow, what would break?** If the answer is nothing, the level you just described is not a level, it is a name. A real promotion has to change something about the shape of the work, otherwise it is a title change and everyone can see through it. This test kills the classic failure where a senior level is defined as "does more of what a mid does, and also reviews code", because nothing about the team's shape changes when three more people start reviewing code.

**Does the level require being the best in the organization, or the best available in this team?** This is the question that kills the most ladders, and it kills them in the direction of being unmeetable. A level that requires the best judgment in the company is a level nobody on your team will ever reach, so it becomes a decoration, and the real negotiation happens on compensation with the title, which nobody wrote down. Levels are relative to a context, name the context, and they become reachable again.

There is a third, less obvious test. Write the level, then ask someone who is very good at their job to read it and tell you what is missing. If the person who most clearly exceeds the level cannot tell you what is missing, it is probably complete. If they can immediately list four things, you have written a job description for the person you already have, which is a different artifact with a different purpose, and a useful one, but not a level.

---

## A Promotion Is Not a Reward for Delivering

This is the failure that generates the most resentment, and it is the one I have had to correct most often in my own team.

Leading a difficult project is not a promotion. Finishing a quarter at capacity is not a promotion. Being the person who always says yes is not a promotion, and a ladder that quietly rewards these behaviors is a ladder that taught your team the wrong lesson, no matter what the document says.

A promotion is a change in the scope of what you are trusted with, and it should only happen when the work at the larger scope has already happened, repeatedly, and successfully. The order matters. If you promote people into the responsibility, you are gambling on them, and gambling on people is the single most expensive thing a manager does with a budget they do not have.

I say this to my team explicitly, every time I do it, because the alternative reading is always available and it is always flattering. When someone asks me why they were promoted, the honest answer is a list of things they already did for months, not a reward for the last sprint. If the list is empty, I should not be promoting them yet, and I have had to tell people that.

There is a second half to this. If the level above is a real level with real scope, then not being there is not a verdict on the person, it is a description of the work they have been doing. That framing is not sugar-coating, it is accurate, and it is the difference between a career conversation and a morale problem.

---

## The Manager's Job Is to Make It Legible

Publishing the ladder is the easy part. Keeping it true is the actual work, and it is almost entirely on the manager.

In practice that means writing specific, dated feedback about scope and decision rights throughout the year, not saving the whole argument for review season. It means going to calibration with evidence rather than adjectives, and being willing to lose an argument there. It means telling your team when you disagree with a decision that was made, and why, so that the levels stay tied to reality. It means writing the level for the person who is ready before they start interviewing for other jobs, because a ladder nobody believes in until somebody resigns is a retention strategy that only works once.

The part I underestimated as a new manager is repetition. A level document that lives in a repository is not discoverable. It has to come up in one-on-ones, in sprint planning, in the conversation where you say "this is the next thing I want to see from you", and again in the review. Ten minutes, several times a quarter, is the actual cost. Everything else is the document, which is cheap.

---

## Where I Would Start on Monday

If you have no ladder and a team that is asking, the order below is the one I would use, and it takes a few weeks rather than a quarter.

- Pick three levels, not seven. Five or more and nobody can hold them in their head.
- Write each one in terms of scope, ambiguity, and decision rights, with an example of real work at that level from your own team.
- Run the "what would break" test on each, and cut anything that fails.
- Publish it, then say out loud in a team meeting what you are unsure about. Naming the open questions does more for trust than a confident document does.
- In every one-on-one for the next two quarters, talk about where the person sits on the three axes. Not the level, the axes.

The ladder will be wrong in places. That is fine, and it is also the point, because a ladder you revise in front of your team is teaching them that progression is a conversation rather than a verdict. That is the habit you are actually trying to build, and the document is just where it starts.

I have not found a version of this that avoids the hard part, which is that writing levels forces you to say out loud what you actually believe about what makes one engineer better than another. That is uncomfortable work, and there is no way to delegate it. If you have ever tried to build a leveling framework and given up, I suspect the blocker was not the format. It was the conversation underneath it.

---

## References

- [Matteo Bisi, From Delegation to Ownership: How to Keep Engineers Motivated](/posts/from-delegation-to-ownership-how-to-keep-engineers-motivated/)
- [Matteo Bisi, From Senior System Engineer to Team Leader](/posts/from-senior-system-engineer-to-team-leader-journey-leadership-principales/)
- [Matteo Bisi, Engineering Managers Are Your Real Culture](/posts/engineering-managers-culture-cto-force-multiplier/)
- [Will Larson, Staff Engineer](https://lethain.com/staff-engineer/) and [An Engineer's Guide to Growing Senior](https://lethain.com/senior-engineer-growth/)
- [Progression.fyi](https://www.progression.fyi/) for a public view of how levels and compensation are named across companies
