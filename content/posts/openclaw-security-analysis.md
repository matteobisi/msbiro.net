---
title: "ClawdBot → MoltBot → OpenClaw: A Case Study in Confusion Attacks and Security Risks"
date: 2026-01-31T01:44:33Z
tags: [
  "cybersecurity", "supply-chain-security", "ai", 
  "devsecops", "threat-analysis", "open-source"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A comprehensive security analysis of the OpenClaw AI assistant project. Examining three name changes in 10 days as a confusion attack pattern, exposed cloud instances due to misconfiguration, the fake VS Code plugin incident, and the hidden costs of running AI agents on your own API keys. Why I can't use this tool with my real accounts despite being an AI enthusiast."
canonicalURL: "https://www.msbiro.net/posts/openclaw-security-analysis/"
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
    alt: "OpenClaw Security Analysis: Supply Chain Security and Confusion Attacks"
    caption: "A security analysis of OpenClaw's multiple rebranding and operational risks"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## What is ClawdBot/MoltBot/OpenClaw?

For those unfamiliar with the project, **OpenClaw** (formerly MoltBot, previously ClawdBot) is a personal AI assistant platform that integrates with multiple messaging channels including WhatsApp, Telegram, Discord, Slack, Signal, iMessage, and many others. The project is available at [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw) and maintains a website at [openclaw.ai](https://openclaw.ai/).

The tool is designed to be a "local-first, single-user assistant" with capabilities that include shell command execution, filesystem operations, browser automation, and integration with various cloud services. It's essentially a bridge between AI models and your entire digital ecosystem. However, **OpenClaw does not provide model access itself**; users must configure it with their own API keys from providers like Anthropic, OpenAI, or others.

From a technical perspective, the project appears to be well-engineered with a substantial TypeScript codebase (~292,000 lines) and comprehensive test coverage (891 test files). On paper, it sounds like a dream tool for any AI enthusiast or power user looking to automate workflows and integrate AI deeply into their daily operations.

---

## Security Analysis: What the Scans Revealed

To understand the security posture of this project, I conducted a comprehensive security analysis using industry-standard tools. I cloned the repository from GitHub and ran three different security scanners: **GitHub Copilot Security Review**, **Semgrep**, and **Snyk Code**. The results paint an interesting picture.

### The Good: Mature Security Architecture

The code review revealed that OpenClaw actually demonstrates **mature security architecture** in several areas:

- **Prompt Injection Defense**: The codebase includes sophisticated prompt injection mitigation for untrusted external content, with pattern detection for 12+ suspicious patterns like "ignore previous instructions" and "you are now a" attempts.
- **Secrets Management**: Proper file permissions and handling of sensitive credentials.
- **Multi-layered Approval System**: Built-in safeguards for dangerous operations.
- **Audit Tooling**: Comprehensive logging and audit capabilities integrated into the CLI.
- **Active Secret Scanning**: The CI/CD pipeline includes secret scanning mechanisms.

The GitHub Copilot security review actually gave it an **overall security posture rating of "STRONG"** for its intended use case as a local-first, personal assistant.

### The Concerning: High-Risk Design Decisions

However, the same analysis highlighted some **inherent risks** that come with the design:

- **High-Privilege Operations by Design**: Shell execution, filesystem access, and browser automation capabilities create a significant attack surface.
- **Complex Multi-Channel Attack Surface**: With 15+ messaging channel integrations, each represents a potential attack vector.
- **Canvas JavaScript Evaluation**: Even when sandboxed, `eval()` capabilities are inherently risky.
- **External Content Ingestion**: Processing emails, webhooks, and other external inputs increases exposure.

### Findings Summary

**Semgrep** identified **192 code findings**, though most were related to:
- Insecure WebSocket usage (ws:// instead of wss://) in documentation and test files
- Exported activities in the Android app
- Configuration patterns that needed review

**Snyk Code** found primarily **LOW severity issues**, mostly hardcoded secrets in test files (acceptable practice for unit tests).

The tools confirmed what the architecture review suggested: the code itself is reasonably well-written, but the **operational security risk is significant** due to the nature of what this tool does.

---

## The Three-Name Game: A Confusion Attack?

One of the most troubling aspects of this situation isn't in the code; it's in the pattern of behavior. **Changing names three times in just 10 days** is exactly what security professionals might call a **confusion attack** or **namespace confusion**.

**The timeline:**
1. **ClawdBot** (original name)
2. **MoltBot** (first rename)
3. **OpenClaw** (current name at [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw))

This rapid rebranding makes it extremely difficult to:
- **Track the project's history** and reputation
- **Search for security advisories** or vulnerability reports
- **Verify the legitimacy** of plugins, extensions, or related resources
- **Maintain consistent security posture** across ecosystems

### The Confusion in Practice

A perfect example of this confusion: [moltbot.you](https://moltbot.you/) currently exists and points to [github.com/gstarwd/clawbot](https://github.com/gstarwd/clawbot). Is this the former official site? A fork? A scam? A legitimate community effort? **It's nearly impossible to tell**, and this ambiguity is exactly what makes confusion attacks so dangerous.

When multiple repositories, websites, and names exist simultaneously, users cannot reliably determine:
- Which version is the "official" one
- Whether they're downloading legitimate code or a malicious fork
- Which community resources are trustworthy
- Where to report security issues

In supply chain security, this is a red flag. Legitimate open-source projects rarely change their fundamental identity multiple times in such a short window. It raises questions about:
- Why the need for multiple rebrandings?
- What prompted each name change?
- Are there legal or compliance issues being avoided?
- Is this an attempt to distance from previous incidents?

**Rumors suggest that Anthropic may have been behind the first name change** (ClawdBot → MoltBot) due to name similarity with their "Claude" brand. While this could explain the initial rebrand, **it does not justify or explain the second name change** (MoltBot → OpenClaw). This additional rebrand only compounds the confusion and raises further questions about the project's stability and governance.

---

## Misconfiguration and Exposed Cloud Instances

Beyond the code itself, reports have surfaced about **hundreds of exposed OpenClaw instances running on cloud infrastructure**. This is a critical operational security failure that demonstrates the gap between secure code and secure deployments.

The platform's documentation explicitly states that the web interface is intended for **localhost or Tailscale-only access**, yet evidence suggests many users have deployed it with public-facing endpoints. This misconfiguration exposes:

- **OAuth tokens** for services like Anthropic, OpenAI, and GitHub Copilot
- **API keys** for messaging platforms (Telegram, Discord, WhatsApp, etc.)
- **Session transcripts** containing potentially sensitive conversations
- **User data** stored in the OpenClaw directory structure

The root cause appears to be inadequate warnings and guardrails in the deployment process. While the code has "security-first default configurations," users are still deploying it in dangerous ways. This is exacerbated by:
- Lack of startup warnings for public binding
- Insufficient documentation emphasis on security implications
- No built-in checks to prevent internet-facing deployments

---

## The VS Code Plugin Incident

Perhaps the most alarming development was the **fake MoltBot AI coding assistant** that appeared on the [Visual Studio Code marketplace](https://thehackernews.com/2026/01/fake-moltbot-ai-coding-assistant-on-vs.html). This incident demonstrates how the confusion around the project's identity can be exploited.

The fake plugin:
- Impersonated the legitimate project
- Was published on the official VS Code marketplace
- Could have potentially harvested API keys and credentials from unsuspecting users
- Took advantage of the brand confusion during the MoltBot era

This is a textbook supply chain attack vector. When a project changes names frequently, it creates opportunities for attackers to:
- Register similar names on package registries
- Create convincing impostor packages
- Exploit users' uncertainty about which version is legitimate
- Steal credentials through malicious plugins masquerading as official extensions

The plugin marketplace incident validates concerns about the confusion attack pattern and demonstrates real-world exploitation.

---

## The Terms of Service Problem

Here's where things get legally and financially concerning. OpenClaw **does not provide access to AI models**—rather, it **requires you to bring your own API keys** from providers like Anthropic, OpenAI, or others. While the platform is technically model-agnostic and can work with local models (via Ollama or similar solutions), reports from the community consistently indicate that **it only performs well with Anthropic's Claude models**, particularly Claude Sonnet 4.5 or Opus 4.5.

### The Cost Reality

This creates a significant financial burden that users may not fully appreciate:

**You are paying for every API call.** Every message, every file operation, every shell command that the AI assistant processes goes through Anthropic's paid API. The costs can be substantial:
- Anthropic's advanced models (like Claude Opus 4.5) charge per token for both input and output
- An AI assistant with shell access, filesystem operations, continuous availability, and multi-channel messaging integration generates **constant API usage**
- Running this kind of autonomous agent 24/7 could easily cost **hundreds or even thousands of dollars per month** depending on usage patterns

The platform's documentation shows you can configure prompt caching to reduce costs, but the fundamental issue remains: **you're running a persistent AI agent on a consumption-based pricing model.**

### The Local Model Problem

Theoretically, you could avoid these costs by using local open-source models. However:
- Community reports indicate that **local models produce significantly inferior results** for the complex agentic workflows OpenClaw requires
- The tool's advanced capabilities (multi-channel coordination, shell operations, file management) demand the reasoning capabilities of top-tier models
- While models like DeepSeek, Qwen, or GLM-4.7 are improving, they reportedly still lag behind Claude Opus for the nuanced, multi-turn agentic tasks that make OpenClaw useful

So you're effectively locked into expensive cloud-based models if you want the tool to work as advertised.

### Potential ToS Violations

Beyond the financial aspect, there are potential **Terms of Service concerns** with how this tool is used:

1. **Acceptable Use Policy**: Using AI models for autonomous shell execution, filesystem access, and continuous agent operations may push the boundaries of acceptable use policies. While not explicitly prohibited, it's unclear whether Anthropic intended their models to be used for persistent, autonomous system control.

2. **Resale or Redistribution**: If anyone is offering "hosted" OpenClaw instances or "shared" setups, this could violate terms against reselling API access.

3. **Usage Patterns**: The high-volume, persistent nature of an always-on AI assistant might trigger rate limiting or raise flags with providers about unusual usage patterns.

4. **Liability for Agent Actions**: When an AI agent has shell access and filesystem permissions, who is responsible if it takes harmful actions? The ToS likely places all liability on the API key owner.

These aren't hypothetical concerns. **Violating cloud provider ToS can result in**:
- Immediate API key revocation
- Account termination
- Loss of access to all services
- Potential financial liability for any damages
- Being banned from future use of the platform

---

## Why I Can't Use This With My Real Accounts

As someone who is genuinely enthusiastic about AI and automation, I would typically be the target audience for a tool like OpenClaw. The feature set is impressive, the integration possibilities are extensive, and the local-first architecture aligns with privacy principles I value.

**But I cannot in good conscience use this tool with my real personal or professional accounts.** Here's why:

### 1. **Supply Chain Trust is Broken**

The three-name shuffle, combined with the VS Code marketplace incident, has destroyed any baseline trust I might have had. I don't know:
- Who is actually maintaining this project
- What their long-term intentions are
- Whether the code I download matches the repository
- If there are additional dependencies being injected

### 2. **Financial and Legal Liability is Too High**

Connecting my real accounts (GitHub, cloud providers, messaging platforms) to a tool with uncertain ToS implications creates personal liability. The risks include:
- Potentially expensive API bills from persistent agent usage
- API keys could be revoked if usage patterns raise concerns
- Accounts could be terminated
- Loss of access to critical services
- Potential financial or legal consequences

### 3. **Operational Security Cannot Be Guaranteed**

Even if I trust the code today:
- The misconfiguration risks are too high
- The attack surface is too broad
- The consequences of a security breach are catastrophic
- I cannot adequately audit all 15+ messaging channel integrations

### 4. **The Risk-Reward Ratio Doesn't Add Up**

Yes, the tool offers powerful capabilities. But:
- Alternative solutions exist that don't carry these risks
- The potential upside doesn't justify the downside
- My professional reputation and account security are worth more than automation conveniences

---

## Final Thoughts

OpenClaw represents an interesting case study in how **technically sound code can still create unacceptable security and legal risks** when:
- Operational practices are questionable
- Identity and branding are unstable
- Deployment guidance is insufficient
- Business models and ToS compliance are unclear

The irony is that the code itself appears to be reasonably well-engineered with thoughtful security controls for its threat model. But the **surrounding ecosystem—the branding chaos, the plugin incidents, the exposed instances, and the ToS questions—creates a risk profile** that no amount of clean code can overcome.

**For security-conscious individuals and organizations, the lesson is clear**: evaluate not just the code, but the entire context around a project. Supply chain security isn't just about vulnerabilities in dependencies—it's about trust, stability, transparency, and legal compliance.

As much as I appreciate the technical capabilities that tools like this promise, **I'll be waiting for solutions that come with clearer provenance, stable identities, and transparent business models** that don't potentially violate service provider terms.

Until then, my real accounts stay disconnected.

---

## References

- [OpenClaw Official Repository](https://github.com/openclaw/openclaw)
- [OpenClaw Website](https://openclaw.ai/)
- [MoltBot.you](https://moltbot.you/) (unclear legitimacy, points to [github.com/gstarwd/clawbot](https://github.com/gstarwd/clawbot))
- [The Hacker News: Fake MoltBot AI Coding Assistant on VS Code Marketplace](https://thehackernews.com/2026/01/fake-moltbot-ai-coding-assistant-on-vs.html)
- Security scan results: GitHub Copilot Security Review, Semgrep, Snyk Code (January 2026)

---

*Disclaimer: This analysis is based on publicly available information, security scans of the public GitHub repository, and reported incidents. The technical assessment reflects the state of the code at the time of analysis (January 31, 2026).*
