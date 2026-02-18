---
title: "The Challenge of Securing AI Agents: A DevSecOps Perspective"
date: 2026-02-17
tags: [
  "cybersecurity", "devsecops", "ai", "agentic-ai", "mcp", "openclaw",
  "cloud-native", "shadow-ai"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A DevSecOps team leader's reflection on the challenge of securing customers who use AI agents that act like users, and how this connects to spec-driven development and MCP security."
canonicalURL: "https://www.msbiro.net/posts/securing-ai-agents-devsecops-challenge/"
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
    alt: "The Challenge of Securing AI Agents: A DevSecOps Perspective"
    caption: "The Challenge of Securing AI Agents: A DevSecOps Perspective"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

As a DevSecOps Team Leader, my job is to secure customers using modern technologies. Sounds straightforward, right? The reality is far more complex. Every day, I face the challenge of enabling innovation while maintaining security. The rapid adoption of AI has introduced a new dimension to this challenge: **agentic AI assistants** that do not just chat, they act.

This challenge connects directly to something I wrote about recently. In my article on [spec-driven development with GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/), I discussed how structure and governance matter when using AI for coding. The same principle applies here: when AI agents can execute code, access secrets, and operate with user privileges, we need structure and governance more than ever.

A recent article from SentinelOne on [securing AI tools that act like users](https://www.sentinelone.com/blog/how-sentinelone-secures-the-ai-tools-that-act-like-users/) articulates what I have been observing in the field. These tools, like OpenClaw (aka Clawdbot and Moltbot), represent a fundamental shift. They execute code, spawn shell processes, access local files and secrets, call external APIs, and operate with the same privileges as the user account running them.

This is not just another security consideration. This is a paradigm shift.

---

## The New Reality: AI Agents as Privileged Users

Traditional security models were built around human users. We know how to authenticate humans, monitor their activities, and enforce least-privilege access. But AI agents blur these lines. When an AI tool can execute arbitrary code on endpoints, access sensitive files and secrets, make API calls to external services, and operate continuously without human oversight, it effectively becomes a **privileged automation account** with all the associated risks and none of the traditional human accountability mechanisms.

Organizations are already experiencing breaches tied directly to unsanctioned AI usage, often at significantly higher costs than traditional incidents. This is not theoretical. It is happening now.

---

## What SentinelOne's Approach Teaches Us

SentinelOne recently published a detailed breakdown of how they secure agentic AI assistants like OpenClaw. Rather than repeating their technical analysis here, I recommend reading their article directly for the implementation details. What I want to highlight is the strategic insight: **effective protection requires three reinforcing control planes** working together.

First, you need **endpoint visibility** through EDR/XDR to detect agent processes, file activity, and network communications. Second, you need **AI interaction security** to govern the point where sensitive data meets AI, including shadow AI discovery and MCP tool-chain governance. Third, you need **agent hardening** within the runtime itself, verifying skill integrity and enforcing zero-trust principles.

This three-layer approach resonates with what I discovered in my own [security analysis of OpenClaw](/posts/openclaw-security-analysis/). Even when the code itself demonstrates mature security architecture, the operational risks remain significant. The combination of high-privilege operations, broad attack surfaces, and deployment misconfigurations creates a risk profile that technical controls alone cannot fully address.

---

## The MCP Connection

In my previous article on [securely using third-party MCP servers](/posts/securely-using-third-party-mcp-servers/), I discussed the security challenges introduced by the Model Context Protocol. The connection between MCP servers and agentic AI assistants is direct and critical.

Agentic AI tools like OpenClaw frequently interact with MCP servers to extend their capabilities. This creates a compound risk: not only do you have an autonomous agent executing code with user privileges, but that agent is also invoking external tools through MCP connections.

The OWASP GenAI security recommendations I discussed earlier, trust minimization, sandbox execution, just-in-time access, and governance workflows, apply doubly when dealing with agentic AI. You need to verify the origin of MCP servers before connecting, pin versions, maintain manifests of approved tools, and establish formal governance workflows. Without these controls, you are essentially giving autonomous agents the keys to your kingdom.

---

## Closing the Governance Gap

Here is the uncomfortable truth: most organizations lack meaningful governance controls to manage AI risk. The gap between those that *believe* they have AI governance and those that *actually do* is exactly where breaches happen.

Following SentinelOne's recommended approach, we need to close this gap through a phased timeline:

* **Immediate (this week)**: Run detection queries to discover agentic AI activity, audit browser extensions, and review AI acceptable use policies to ensure they address autonomous agents
* **Short-term (90 days)**: Move from inventory to continuous visibility, establish sanctioned alternatives for employees, and operationalize behavioral detection as automated rules
* **Long-term (6 months)**: Complete full AI tool inventory with risk scoring, establish enforcement policies at endpoint and network layers, and build board-ready reporting metrics

The key principles to remember:

1. **AI agents are privileged users**: Treat agentic AI assistants as privileged automation accounts that execute code and access secrets with real system privileges
2. **Three-layer defense is essential**: Effective security requires EDR/XDR, AI interaction security, and agent hardening working together
3. **MCP security compounds risk**: When agents interact with MCP servers, apply OWASP guidance on trust minimization, sandboxing, and governance workflows
4. **Visibility precedes governance**: You cannot govern what you cannot see; discovery is the essential first step
5. **Structure enables safe adoption**: Just as spec-driven development brings governance to AI coding, structured security frameworks enable safe AI agent adoption
6. **Security enables AI adoption**: Security is not the department that says no to AI. It is the function that makes AI possible at enterprise scale.

---

## Conclusion

Securing modern technologies is not about blocking innovation. It is about building the frameworks that allow innovation to thrive safely. Agentic AI assistants represent a significant leap forward in productivity, but they also introduce risks that traditional security models were not designed to handle.

As DevSecOps leaders, our challenge is to adapt our security posture to this new reality. This means moving from monitoring what employees type into a browser to governing what autonomous processes do on our endpoints. It means understanding the intersection of AI agents and external tool chains like MCP. And it means applying the same principles of structure and governance that we use for AI-assisted development to AI agent security.

The organizations that succeed will not be those that adopted AI fastest or blocked it longest. They will be the ones that built the security foundations to adopt it safely.

---

## References

* SentinelOne: [Shadow Agents: How SentinelOne Secures the AI Tools That Act Like Users](https://www.sentinelone.com/blog/how-sentinelone-secures-the-ai-tools-that-act-like-users/) (for detailed technical implementation of the three-layer defense)
* My Previous Article: [Securely Working with Third-Party MCP Servers](/posts/securely-using-third-party-mcp-servers/)
* My Previous Article: [GitHub Spec-Kit: Why Structured AI Development Beats Vibe Coding](/posts/github-spec-kit-spec-driven-development/)
* My Previous Article: [ClawdBot → MoltBot → OpenClaw: A Case Study in Confusion Attacks and Security Risks](/posts/openclaw-security-analysis/)
* OWASP GenAI Project: [MCP Security Cheatsheet](https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/)
