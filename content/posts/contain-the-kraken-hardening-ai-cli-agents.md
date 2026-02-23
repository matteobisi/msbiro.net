---
title: "Contain the Kraken: Hardening AI CLI Agents for Enterprise-Grade DevSecOps"
date: 2026-02-21T14:00:00Z
tags: [
  "cybersecurity", "devsecops", "AI", "automation",
  "claude-code", "gemini-cli", "github-copilot", "sandboxing", "trail-of-bits"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "As AI agents gain the power to execute code and manage infrastructure, the risk of unmanaged access grows. Inspired by Trail of Bits, this guide explores how to harden AI CLI tools like Claude Code, Gemini, and Copilot using sandboxing and zero-trust principles."
canonicalURL: "https://www.msbiro.net/posts/contain-the-kraken-hardening-ai-cli-agents/"
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
    alt: "Hardening AI CLI Agents"
    caption: "Moving from 'vibe coding' to secure, agentic workflows"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction: From Secure Specs to Secure Agents

Every day, as a DevSecOps Team Leader, I wake up with the same exciting challenge: how do we maintain an "enterprise grade" of work in a landscape shifting beneath the feet of AI? For me, this isn't just about checking boxes; it’s about the daily discipline of ensuring that every tool we introduce into our workflow meets the same rigorous standards we apply to our production code.

In my previous articles, I’ve advocated for **Spec-Driven Development (SDD)** using tools like [GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/) and OpenSpec. We discussed how structured specifications serve as a "Constitution" for our projects—providing embedded documentation, clear guardrails, and the vital audit trails necessary for accountability and compliance (especially under frameworks like NIS2 or ISO 27001). 

But there is a missing piece to this puzzle. 

Even with the most perfect specifications, if the clients we use to implement them—powerful "agentic" tools like **Gemini CLI**, **GitHub Copilot**, or the high-velocity **Claude Code**—are running directly on our host machines with full filesystem and network access, we are effectively handing a loaded gun to a black box. If we want an enterprise-grade workflow, we need more than just secure *logic*; we need a secure *environment*.

This is where the industry's heavy hitters are stepping in. **[Trail of Bits](https://www.trailofbits.com/)**—a world-renowned security research and consulting firm known for auditing the most critical software on the planet—has recently released a blueprint for this exact problem. Through their work on [claude-code-config](https://github.com/trailofbits/claude-code-config) and [claude-code-devcontainer](https://github.com/trailofbits/claude-code-devcontainer), they’ve demonstrated how to "trap" an AI agent in a hardened, zero-trust sandbox. 

It is time we take these lessons and apply them to the entire "Big Three" of AI CLI tools.

---

## The Hardening Matrix: Gemini vs. Copilot vs. Claude

To achieve an enterprise grade of work, we have to look at how these tools handle three critical security pillars: **Filesystem Isolation**, **Network Guardrails**, and **Auditability**. 

| Security Feature | **Claude Code** (Hardened) | **Gemini CLI** | **GitHub Copilot CLI** |
| :--- | :--- | :--- | :--- |
| **Primary Sandbox** | Native (Seatbelt/Bwrap) + Devcontainer | OS Process (Standard) | GitHub Codespaces (Cloud-only) |
| **Filesystem Access** | Restricted to project root via `settings.json` | Full user access (unless manually containerized) | Isolated to VM (if using Codespaces) |
| **Command Guardrails** | Configurable "Hooks" to block `rm -rf`, `git push --force` | None (Standard shell permissions) | Confirmation prompts for most commands |
| **Audit/Logging** | Bash command logs + prompt history in `CLAUDE.md` | Session logs | Proprietary telemetry |
| **Network Control** | Allow-listing via `iptables` in Devcontainer | Open outbound access | Open outbound access |
| **"Bypass" Safety** | Safe in sandbox (Trail of Bits model) | Risky (Manual confirmation recommended) | N/A (Manual confirmation required) |

---

## The Solution: Multi-Layered Defense

### Layer 1: The Runtime Sandbox (Devcontainers)
The most effective way to secure an agent is to ensure it never sees your host machine. Inspired by the `claude-code-devcontainer` approach, you should run your AI CLI inside a Docker container where only the project files are mounted. This "traps" the agent, preventing it from accessing your `~/.ssh` keys, `~/.aws/credentials`, or your shell history.

### Layer 2: Configuration as Guardrails
Hardening isn't just about where the tool runs, but what it's allowed to do. For Claude Code, this means using a `settings.json` that explicitly denies access to sensitive paths. For Gemini and Copilot, it means wrapping the execution in a shell environment that restricts environment variables and enforces specific permissions.

### Layer 3: The Role of the 'Constitution'
Connecting back to my work with **AGENTS.md** and **Spec-Kit**, we must treat security as a specification. By defining "Security Boundaries" in your context files, you give the AI agent a set of rules to follow. However, as the Trail of Bits research shows, these are *guidelines*, not *boundaries*. True security must be enforced by the OS and the container runtime.

---

## Practical Walkthrough: Hardening the Gemini CLI

While Claude Code has built-in sandboxing hooks, we can apply the same "Zero-Trust" principles to the **Gemini CLI** using a simple Devcontainer setup.

### 1. Create a `devcontainer.json`
Configure your environment to be "Non-Root" and restrict the workspace.

```json
{
  "name": "Hardened AI Workspace",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:20",
  "remoteUser": "node",
  "mounts": [
    "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=cached"
  ],
  "customizations": {
    "vscode": {
      "extensions": ["google.gemini-vsc"]
    }
  }
}
```

### 2. Enforce Network Egress
If you are using a tool like `cilium` or `iptables` inside your container, you should restrict the agent's network access to only the necessary API endpoints:
*   `generativelanguage.googleapis.com` (for Gemini)
*   `api.anthropic.com` (for Claude)

---

## Conclusion: Security Enables Innovation

As I've said before, the goal of DevSecOps is not to say "no" to new tools. It is to find the "yes" that doesn't put the company at risk. By moving from "Vibe Coding" to **Spec-Driven Development** and then further to **Hardened Agentic Workflows**, we can finally harness the full speed of AI without losing our sleep—or our credentials.

Special thanks to the team at **Trail of Bits** for their pioneering work in this space. Their repositories are a must-read for anyone serious about the future of AI in the enterprise.

**Next Steps:**
*   Review your `~/.bash_history` for leaked API keys used in CLI tools.
*   Experiment with running your next `claude` or `gemini` session inside a Docker container.
*   Update your `AGENTS.md` with explicit security boundaries.

Stay secure, stay automated!
