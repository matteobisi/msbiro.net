---
title: "Enterprise-Grade Hardening for AI CLI Agents"
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
description: "As AI agents gain the power to execute code and manage infrastructure, the risk of unmanaged access grows. This guide explores how to reach an enterprise grade of security by hardening AI CLI tools like Gemini, Copilot, and Claude Code using sandboxing and zero-trust principles."
canonicalURL: "https://www.msbiro.net/posts/hardening-ai-cli-agents-enterprise-grade/"
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

## Introduction: Reaching an Enterprise Grade of Security

Every day, as a DevSecOps Team Leader, I wake up with the same exciting challenge: how do we maintain an \"enterprise grade\" of work in a landscape shifting beneath the feet of AI? For me, this isn't just about checking boxes; it’s about the daily discipline of ensuring that every tool we introduce into our workflow meets the same rigorous standards we apply to our production code.

In my previous articles, I’ve advocated for **Spec-Driven Development (SDD)** using tools like [GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/) and OpenSpec. We discussed how structured specifications serve as a "Constitution" for our projects—providing embedded documentation, clear guardrails, and the vital audit trails necessary for accountability and compliance (especially under frameworks like NIS2 or ISO 27001). 

But there is a missing piece to this puzzle. 

Even with the most perfect specifications, if the clients we use to implement them—powerful "agentic" tools like **Gemini CLI**, **GitHub Copilot**, or the high-velocity **Claude Code**—are running directly on our host machines with full filesystem and network access, we are effectively handing a loaded gun to a black box. If we want an enterprise-grade workflow, we need more than just secure *logic*; we need a secure *environment*.

This is where the industry's heavy hitters are stepping in. **[Trail of Bits](https://www.trailofbits.com/)**—a world-renowned security research and consulting firm known for auditing the most critical software on the planet—has recently released a blueprint for this exact problem. Through their work on [claude-code-config](https://github.com/trailofbits/claude-code-config) and [claude-code-devcontainer](https://github.com/trailofbits/claude-code-devcontainer), they’ve demonstrated how to "trap" an AI agent in a hardened, zero-trust sandbox. 

It is time we take these lessons and apply them to the entire "Big Three" of AI CLI tools.

---

## The Hardening Matrix: Gemini vs. Copilot vs. Claude

To achieve an enterprise grade of work, we have to look at how these tools handle three critical security pillars: **Filesystem Isolation**, **Network Guardrails**, and **Auditability**. 

Looking at my local machine, I can see these agents already have their own "roots": `~/.copilot` (containing `config.json`) and `~/.gemini` (with `settings.json` and `GEMINI.md`). These are our first hooks for hardening.

| Security Feature | **Claude Code** (Hardened) | **Gemini CLI** | **GitHub Copilot CLI** |
| :--- | :--- | :--- | :--- |
| **Primary Sandbox** | Native (Seatbelt/Bwrap) + Devcontainer | **Native (macOS Seatbelt)** + Docker/Podman | GitHub Codespaces or Isolated VM |
| **Filesystem Access** | Restricted via `settings.json` | Restricted via **`excludeTools` / `coreTools`** | Isolated to VM / Workspace |
| **Command Guardrails** | `CLAUDE.md` + manual confirmation | **`GEMINI.md` + YOLO Mode (Off)** | **Enterprise Policies** (Auto-approve Off) |
| **Audit/Logging** | Bash logs + `CLAUDE.md` | Session logs + Checkpoints | **Advanced Audit Logging** (Enterprise) |
| **Network Control** | `iptables` in Devcontainer | Proxy routing + API Allowlisting | Enterprise Proxy / Firewall |
| **Policy Enforcement** | JSON Configuration | **Centralized System Settings** | GitHub Enterprise Policies |

---

## The Solution: Multi-Layered Defense

### Layer 1: Runtime Isolation (Seatbelt & Containers)
The most effective way to secure an agent is to ensure it never sees your host machine. 
*   **Claude Code** and **Gemini CLI** both offer native sandboxing on macOS via **Seatbelt**. This restricts the agent's ability to read your `~/.ssh` or `~/.aws` folders by default.
*   **For Enterprise Grade:** Wrap the execution in a Docker/Devcontainer. This "traps" the agent in a virtual environment where only the project files are mounted.

### Layer 2: Configuration as Guardrails
Hardening isn't just about where the tool runs, but what it's allowed to do. 

#### Claude Code (The Blueprint)
The **Trail of Bits** repository for [claude-code-config](https://github.com/trailofbits/claude-code-config) provides a masterclass in this approach. Their [settings.json](https://github.com/trailofbits/claude-code-config/blob/main/settings.json) uses a `permissions.deny` block to explicitly forbid:
*   **Destructive Commands:** `rm -rf`, `sudo`, `mkfs`.
*   **Sensitive Reads:** `~/.ssh/**`, `~/.aws/**`, `~/.kube/**`, and even crypto wallets.
*   **Remote Execution:** `curl *|bash*`.
*   **Forced Operations:** `git push --force`.

By treating the AI agent's configuration as a security policy, they ensure that even if the agent is "convinced" to act maliciously, the underlying toolchain will block the action.
*   **Ref:** [Claude Code Securely deploying AI agents](https://platform.claude.com/docs/en/agent-sdk/secure-deployment)

#### Gemini CLI (The Enterprise Contender)
Following a similar philosophy, the **Gemini CLI** uses its own `~/.gemini/settings.json` (or project-level `.gemini/settings.json`) to enforce boundaries.
*   **Tool Filtering:** Use `excludeTools` to block high-risk capabilities like `run_shell_command` or `write_file` in sensitive contexts.
*   **Seatbelt Integration:** On macOS, Gemini can leverage native **Seatbelt** profiles to sandbox the process at the kernel level.
*   **Ref:** [Gemini CLI Configuration Guide](https://geminicli.com/docs/reference/configuration/)

#### GitHub Copilot (The Managed Approach)
For **GitHub Copilot**, the hardening shift is managed at the organizational level through **GitHub Enterprise Policies**.
*   **Terminal Auto-Approve:** Ensure `terminal_auto_approve` is disabled in `~/.copilot/config.json`.
*   **Code Referencing:** Enable filters to detect and suppress suggestions matching public code.
*   **Ref:** [GitHub Trust Center](https://github.com/trust-center)

### Layer 3: The Role of the 'Constitution'
Connecting back to my work with **AGENTS.md** and **Spec-Kit**, we must treat security as a specification. 
*   **Gemini CLI** uses `GEMINI.md`.
*   **Claude Code** uses `CLAUDE.md`.
Both files act as a "Constitution" for the agent. By defining "Security Boundaries" in these files, you give the AI a set of rules to follow. However, as the **Trail of Bits** research shows, these are *guidelines*, not *hard boundaries*. True security must be enforced by the OS (Seatbelt/Bwrap) and the container runtime.

---

## Practical Walkthrough: Hardening the Gemini CLI

While Claude Code has built-in sandboxing hooks, we can apply the same "Zero-Trust" principles to the **Gemini CLI** using its native configuration options in `~/.gemini/settings.json`.

### 1. Disable 'YOLO' Mode
By default, some agents might execute commands without asking. For an enterprise grade of work, **explicit confirmation** is non-negotiable.

```json
{
  "yolo": false,
  "coreTools": ["read_file", "grep_search", "list_directory"],
  "excludeTools": ["run_shell_command", "write_file"]
}
```
*Note: This configuration forces the agent to ask for permission before performing any file modification or shell execution.*

### 2. Leverage macOS Seatbelt
If you're on macOS, you can run the agent with **Seatbelt** enabled. This uses the same underlying technology as Docker but natively in the OS, preventing the process from accessing unauthorized paths.

### 3. Creating the 'Constitution' (GEMINI.md)
Just like `CLAUDE.md`, you can create a `GEMINI.md` at the root of your project. This file should contain:
*   **Coding Standards:** (e.g., "Always use TypeScript for new files").
*   **Security Boundaries:** (e.g., "Do not ever modify the `.env` file").
*   **Enterprise Workflows:** (e.g., "Run tests before suggesting a code change").

### 4. Comparison: GitHub Copilot CLI Hardening
For **GitHub Copilot**, the hardening moves from the individual file to the **Enterprise Console**. 
*   **Admin Control:** Enforce a "Block Suggestions matching Public Code" policy for all developers.
*   **Agent Control:** In the `~/.copilot/config.json`, ensure that `terminal_auto_approve` is always set to `false`.
*   **Auditability:** Use the "Advanced Audit Logs" in your GitHub Enterprise settings to track every prompt and suggestion.

---

## Conclusion: Security Enables Innovation

As I've said before, the goal of DevSecOps is not to say "no" to new tools. It is to find the "yes" that doesn't put the company at risk. By moving from "Vibe Coding" to **Spec-Driven Development** and then further to **Hardened Agentic Workflows**, we can finally harness the full speed of AI without losing our sleep—or our credentials.

By implementing these multi-layered defenses—from **Seatbelt** sandboxing on macOS to **GEMINI.md** constitutions and **Enterprise Policies** in Copilot—we reach an enterprise grade of work that is ready for production.

Special thanks to the team at **Trail of Bits** for their pioneering work in this space. Their research on `claude-code-config` is a must-read for anyone serious about the future of AI in the enterprise.

**Next Steps:**
*   **Audit Your Configs:** Check `~/.gemini/settings.json` and `~/.copilot/config.json` for auto-approve settings.
*   **Create Your Constitution:** Add a `GEMINI.md` or `CLAUDE.md` to your project root with your coding and security guidelines.
*   **Explore Native Sandboxing:** Test running your AI sessions with macOS Seatbelt or Docker isolation.

Stay secure, stay automated!
