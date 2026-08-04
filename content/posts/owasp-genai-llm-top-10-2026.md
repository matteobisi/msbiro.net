---
title: "OWASP GenAI LLM Top 10 2026: What the New Rankings Mean for Security Teams"
date: 2026-08-04T09:00:00Z
tags: [
  "owasp", "genai", "llm", "ai-security", "cybersecurity",
  "appsec", "prompt-injection", "supply-chain", "devsecops",
  "vulnerability-management", "threat-modeling"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "OWASP released the GenAI LLM Top 10 2026, the first edition grounded in 7,714 real AI security incidents. Prompt Injection stays at number one. Excessive Agency climbs. Misinformation is the widest gap between what practitioners fear and what the evidence shows."
canonicalURL: "https://www.msbiro.net/posts/owasp-genai-llm-top-10-2026/"
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
    alt: "OWASP GenAI LLM Top 10 2026"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

The OWASP GenAI LLM Top 10 2026 was published on August 3, 2026. I am still learning my way around AI security, and I read OWASP Top 10 lists because they are the closest thing the industry has to a consensus map. They are not academic papers. They become the basis for threat models, security review checklists, and procurement questions. The GenAI edition specifically maps the risks that matter when you build applications around large language models.

The 2026 edition is different in one important way. Earlier versions were built on expert judgment. Hundreds of practitioners voted on what they thought were the biggest risks, and that vote built the list. This year, the project team cross-checked that judgment against 7,714 real AI security incidents pulled from public vulnerability databases and an AI-harm database. The community vote still carries 75 percent of the ranking weight, but the remaining 25 percent comes from what has actually happened. The disagreements between belief and evidence are where the most useful lessons sit.

---

## The 2026 Rankings

| Rank | Entry | Change from 2025 |
| --- | --- | --- |
| 1 | Prompt Injection | Unchanged |
| 2 | Sensitive Information Disclosure | Unchanged |
| 3 | Excessive Agency | Moved up |
| 4 | Supply Chain | Renamed and expanded |
| 5 | Data and Model Poisoning | Moved up |
| 6 | Unbounded Consumption | Up 4 places |
| 7 | Misinformation | Widest gap with evidence |
| 8 | Hidden Context Exposure | Renamed (was System Prompt Leakage) |
| 9 | Vector and Embedding Weaknesses | Expanded scope |
| 10 | Improper Output Handling | Down 5 places |

---

## What Stood Out

**Prompt Injection is still number one, and the reason is worth understanding.** If you rank the risks by incident data alone, prompt injection does not even place in the top ten. That is not because it is safe. It is because teams that run LLMs in production fight injection so hard that few clean exploits make it into public reports. The data understates the risk. The people doing the work know the surface is everywhere a model reads untrusted input, so the community vote keeps it at the top.

**Excessive Agency climbed to third.** This is the risk that an LLM given tools and permissions can be tricked into using them in harmful ways. Both the community vote and the incident data agree this is where real damage happens. The report's message is direct: giving an LLM access to file systems, APIs, or email without limiting its permissions and requiring human approval for important actions is no longer a design oversight. It is a top-three vulnerability.

**Misinformation is the widest gap between what people worried about and what actually happened.** Experts ranked it low. The incident data ranked it high. When a model produces fluent, confident output that drives a real decision or action, a wrong answer becomes a wrong outcome. The 2026 edition moved Misinformation up because the evidence demanded it. This is the entry to pay attention to if your system acts automatically on model output without checking it first.

**Unbounded Consumption rose four places.** A single prompt can now trigger expensive reasoning chains, repetitive tool calls, or extended processing that runs up a significant cloud bill. The standard defense of limiting requests per second does not help when the cost is per-token rather than per-request. If you are paying a provider for every token an agent processes, this entry is directly relevant.

**System Prompt Leakage was renamed to Hidden Context Exposure.** The old name was too narrow. The risk is not only about leaking the system prompt. It is about everything the model sees that the user should not: tool descriptions, role definitions, safety rules, formatting instructions. The principle is simple: assume everything in the model's context is discoverable. Never put credentials there. Never rely on its secrecy to control what the model can do.

**Improper Output Handling fell to tenth place.** This covers the basic security practice of validating what a model returns before passing it to another system. Its drop does not mean the risk went away. It means more teams now treat output validation as standard application security, which is where it belongs.

Beyond the rankings, several entries absorbed related risks rather than creating new categories.

Prompt Injection now covers attacks where instructions are hidden inside images or audio, not just typed as text. Supply Chain now covers the risk of pulling a model from a public registry by name, only to have that name taken over and replaced with a malicious copy. Data and Model Poisoning now covers attacks that change model behaviour by modifying configuration files, without touching the model itself.

---

## A Line Worth Drawing

The report draws a clear boundary that matters more as LLMs are given more capabilities. The GenAI LLM Top 10 covers the risk when the model is a component inside your application. The moment that model starts acting on its own, with tools it can call and actions it can take, the risk moves to the companion [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/initiatives/agentic-security-initiative/). Many real incidents sit exactly on this line. If your model makes decisions and acts on them, read both lists.

---

## What I Take From This

I am a DevSecOps team leader learning about AI security, not an AI security specialist. Here is what I am taking away from this report.

First, the report's authors are clear that prompt injection cannot be reliably prevented at the model level. The defense has to be in the system around the model. Build as if injection will succeed, and limit what a compromised model can reach and what its outputs can do.

Second, giving an LLM tools and autonomy multiplies every risk on the list. A bad chat response is one thing. A bad shell command or API call is another. The report is not saying avoid agents. It is saying scope their permissions before they ship and require human approval for anything with real impact.

Third, the supply chain risks that traditional application security teams worry about now apply to AI artifacts too. Models pulled from public registries, third-party components that modify model behaviour, and AI-suggested package names that attackers have pre-registered are all documented threats. The same discipline we apply to container images and dependencies now extends to the models themselves.

None of this requires deep AI expertise to act on. A team that already practices least privilege, input validation, and supply chain hygiene has the right instincts. The GenAI Top 10 tells you where to apply them.

---

## References

- [OWASP GenAI LLM Top 10 2026 (full PDF and resource page)](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
- [OWASP GenAI Security Project](https://genai.owasp.org/)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/initiatives/agentic-security-initiative/)
- [OWASP GenAI Data Security 2026 (DSGAI)](https://genai.owasp.org/initiatives/ai-data-security-initiative/)
- [MITRE ATLAS (Adversarial Threat Landscape for AI Systems)](https://atlas.mitre.org/)
- [NIST AI 600-1 Generative AI Profile](https://www.nist.gov/artificial-intelligence/ai-600-1)
- [EU AI Act (Regulation 2024/1689)](https://eur-lex.europa.eu/eli/reg/2024/1689)
