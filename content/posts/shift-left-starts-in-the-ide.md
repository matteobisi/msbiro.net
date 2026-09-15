---
title: "Shift Left Starts in the IDE: VS Code Security in 2026"
date: 2026-09-15T05:05:55+01:00
tags: [
  "cybersecurity", "devsecops", "developer-security",
  "supply-chain-security", "vscode", "shift-left", "malware", "threat-intelligence"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Learn how to harden VS Code against malicious tasks and extensions, use Workspace Trust safely, and inspect untrusted repositories before opening them."
keywords:
  - "VS Code security"
  - "malicious VS Code tasks"
  - "VS Code Workspace Trust"
  - "IDE security"
  - "developer workstation security"
canonicalURL: "https://www.msbiro.net/posts/shift-left-starts-in-the-ide/"
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
    alt: "A VS Code window open on a cloned repository, with a hidden tasks.json ready to run on folder open."
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

I have been preparing talks for [DevSecOps Day](https://www.devsecopsday.it/) in Bologna this October, and I keep landing on the same sentence: if your shift left story starts at the pipeline, you are already late. In most orgs, shift left still means moving SAST and SCA earlier in CI. Signed images and policy as code are useful. None of them help if the first credential theft happened in the editor, on a laptop, while someone was trying to be a good engineer.

The developer workstation is now an identity store. GitHub PATs, `~/.ssh`, `~/.aws`, cloud CLIs, browser cookies, agent configs, and sometimes a hardware wallet adjacent to the same user account. Compromise the editor session and you skip the scanner and the admission controller. You become the pipeline.

2026 is shaping up as a year where developers are not collateral damage. They are the target. The mission is boring and effective: steal tokens, SSH keys, cloud credentials, wallet files, and then use that identity to move into the rest of the supply chain. I do not think this year will be remembered for a new class of bug. I think it will be remembered as the year enough people finally treated the IDE as production-adjacent. Attackers already did. I wrote about this earlier when [Lazarus started hiding loaders in git hooks](/posts/lazarus-group-git-hooks-malware-developers/). The campaign did not stop. It moved into the IDE.

---

## Check VS Code tasks before opening a repository

**Jenn Gile**, co-founder of [Open Source Malware](https://opensourcemalware.com/), shared a simple check that people skip because it looks too cheap to be real.

If you are going to fork a repo or consume a package, do this before you open the folder in VS Code:

1. Open `.vscode/tasks.json`
2. Look for `./public/fonts/fa-solid-400.woff2` (or another "font" under `public/fonts/`)
3. Look for `"runOn": "folderOpen"`

That is it. One of the most common infection vectors they are seeing right now is a poisoned `tasks.json` that autoruns a fake font file. The file looks like Font Awesome. It is JavaScript. Node executes it. You do not need a malicious extension. You do not even need the rest of the repo to be JavaScript. The editor runs the task because you trusted the folder.

The durable write-up is here: [DPRK Contagious Interview campaign, fake fonts, malicious VS Code tasks](https://opensourcemalware.com/blog/dprk-contagious-interview-campaign-fake-font-uses-malicious-vs-code-fonts).

---

## What VS Code can run when you trust a workspace

VS Code tasks are a legitimate feature. They live in `.vscode/tasks.json`, they are committed with the repo, and they can run linters, compilers, and test harnesses. They can also run on folder open.

A typical malicious task looks like this:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "eslint-check",
      "type": "shell",
      "command": "node",
      "args": ["${workspaceFolder}/public/fonts/fa-solid-400.woff2"],
      "presentation": {
        "reveal": "never",
        "echo": false,
        "close": true
      },
      "runOptions": {
        "runOn": "folderOpen"
      }
    }
  ]
}
```

Read it the way an attacker designed it. The label looks like hygiene. `reveal: never` hides the terminal. `runOn: folderOpen` means the payload starts when the workspace opens, not when you press a key. The "font" is not a font. `file` on a real sample returns JavaScript with a very long line, not a `wOF2` header.

From there the chain is familiar if you have followed Contagious Interview: obfuscated JavaScript (BeaverTail in several variants), a callback that pretends to be a crypto API, then **InvisibleFerret** as a Python backdoor. The goal is credential theft and persistence, not a funny popup.

{{< mermaid >}}
flowchart TB
  A[Fake recruiter or poisoned repo] --> B[Clone and open in VS Code]
  B --> C{Workspace Trust}
  C -->|Restricted Mode| D[Tasks blocked]
  C -->|Trust or parent already trusted| E[folderOpen task]
  E --> F[node runs fake .woff2]
  F --> G[Loader and C2]
  G --> H[Tokens, keys, wallets]
{{< /mermaid >}}

Microsoft already documented this class of risk in [Workspace Trust](https://code.visualstudio.com/docs/editing/workspaces/workspace-trust). Restricted Mode disables tasks, debugging, a chunk of workspace settings, and untrusted extensions. That control is real. The campaign still works because the social engineering is aimed at the moment after the prompt: a coding assessment, a "quick look at this fork", a repo under a parent folder you trusted months ago.

If you trusted `~/src` or `~/Downloads`, every new clone under that tree inherits trust. Restricted Mode never appears. The task runs.

---

## VS Code security hardening checklist

I am not telling you to uninstall it. VS Code is still the default editor in a huge number of teams, including teams I work with. The question is whether you run it like a text editor or like an execution environment, because it is the second one.

Do this in **User Settings**, not in `.vscode/settings.json` of a random repo. Workspace settings are part of the attack surface.

### 1. Keep Workspace Trust on, and stop trusting parent trees

```json
{
  "security.workspace.trust.enabled": true,
  "security.workspace.trust.untrustedFiles": "prompt"
}
```

Open **Workspaces: Manage Workspace Trust** and audit the list. Remove home directories, `Downloads`, and any catch-all `~/git` or `~/src` you use for interview tests and forks. Trust specific repos you actually own. When in doubt, leave Restricted Mode on and read the files first.

### 2. Turn automatic tasks off globally

```json
{
  "task.allowAutomaticTasks": "off"
}
```

That setting is the control that matches Jenn's IoC. `folderOpen` should not be a silent feature on a machine that holds production credentials. If a project needs a task, run it yourself after you have read `tasks.json`.

Do not assume the default saved you. Set it explicitly. Confirm it in User Settings JSON.

### 3. Inspect `.vscode` before you trust the folder

A 20-second check from the terminal, before `code .`:

```bash
# Any autorun task?
rg -n "runOn|folderOpen" .vscode/tasks.json .vscode/*.json 2>/dev/null

# Fake font / node-on-binary pattern
rg -n "fonts/.*woff|\.woff2|node .*public" .vscode/tasks.json 2>/dev/null

# Executables hidden as assets
file public/fonts/* 2>/dev/null
```

If `file` says a `.woff2` is JavaScript, stop. Treat the machine as a potential incident if you already opened the folder in a trusted window.

### 4. Treat Restricted Mode as a review mode, not an annoyance

In Restricted Mode, VS Code is trying to prevent automatic execution. That is the correct posture for a repo you did not write. Do not "enable trust so IntelliSense works" as a reflex. Read `tasks.json`, `launch.json`, `.vscode/settings.json`, and any path that points at a binary.

Also remember: Workspace Trust cannot save you from a malicious extension that ignores Restricted Mode. Install extensions from publishers you can name. Prefer allowlists in managed environments.

### 5. Open unknown code in a disposable environment

The same advice I gave for git hooks still applies. A coding test, a random fork, a "please review this package" should not land next to `~/.ssh`. You do not need a paid VDI for this. Two tools I have already used on real machines are free, including in a corporate laptop policy that already made Docker Desktop a license argument.

The cheapest inspection, if you already have a Linux runtime and you only want to *read* the tree:

```bash
# No network, throw-away container, repo mounted read-only
docker run --rm -it --network none -v "$PWD":/src:ro ubuntu:24.04 bash
```

Do not mount `$HOME`. Do not copy `.env` files in "to make it work". If Node has to run, you have left inspection and you need a stronger boundary.

**Docker Sandboxes (`sbx`)** is that boundary when the take-home will actually execute. I wrote a [hands-on on Docker Sandboxes](/posts/docker-sandboxes-ai-agents/). It is a standalone CLI. It does not require Docker Desktop, so it is usable in shops that blocked Desktop on licensing and still need a local Linux VM. Each session gets its own microVM, its own kernel, and a network policy you can set to locked down or balanced. Clone the assessment inside the sandbox. Keep host credentials on the host. If an autorun task or an agent goes loud, you delete the sandbox.

**Apple Container** is the other free path on macOS. [Apple's open-source container runtime](/posts/apple-container-oss-macos26/) runs an OCI Linux container in a lightweight VM, one VM per container, with no Docker Desktop conversation. I have [tested container machine](/posts/apple-container-1-container-machine-hands-on/) for isolation. Same rules: clone inside the VM, do not bind-mount your home directory, delete the container when you are done.

The rule is the same for all three. If the environment dies, you lose a clone, not a cloud account.

### 6. Treat VS Code-compatible editors the same way

VS Code-compatible editors still read `.vscode/tasks.json`. Switching skin does not switch threat model. If a fork auto-trusts workspaces or re-enables automatic tasks, you went sideways, not left.

---

## Alternatives are available, but no tool is safe by default

This is not an argument that everyone should make the same editor choice. In 2026 there are capable open-source, terminal-native, and commercial alternatives. Some are faster, some better fit a team workflow, and some make it easier to see when opening a repository becomes permission to execute its code.

Microsoft is making a real effort. The Visual Studio Marketplace [scans extensions at publication and again after publication](https://developer.microsoft.com/blog/security-and-trust-in-visual-studio-marketplace/), performs recurring rescans, supports verified publishers, and can block and uninstall extensions known to be malicious. Those controls matter.

They do not change the incentive. An extension compromise can reach an enormous, privileged developer audience, so attackers will keep trying typosquats, compromised publisher accounts, delayed payloads, and poisoned dependencies. A verified publisher establishes an identity. It does not prove that every release is safe.

The problem is broader than malicious extensions. Workspace tasks, debug configurations, settings, and trusted parent folders can turn `code .` into execution. Extensions can read workspace files, spawn processes, make network requests, and bring dependencies. Language servers, formatters, debuggers, development containers, package-manager scripts, and build imports deserve the same trust decision. A broad trusted location or a global tool can make that decision before the prompt appears.

Agentic features add another control plane. Before enabling one, understand what it can read, which tools it can invoke, when it can run commands, and whether it can load MCP servers or instructions from the workspace. The [chat pet](https://code.visualstudio.com/docs/agents/reference/chat-pet) that naps and celebrates above the agent input is not the threat. It is, however, a remarkable choice of bedside manner for software that may be approving terminal commands on your repository.

No editor is universally better. Smaller or terminal-native tools can be easier to keep minimal, but their plugins and integrations still execute code. The useful question is whether you can explain what happens when you open an untrusted repository, what grants it more privilege, and how to reverse that decision.

---

## Shift left starts in the IDE

Shift left is not a scanner stage. It is the first place untrusted code meets a privileged identity. In 2026 that place is often the IDE, on a developer laptop, with cloud credentials already loaded.

You can stay on VS Code. Plenty of teams will, especially where established tooling and collaboration needs make a change impractical. Keep your eyes open: disable automatic tasks, stop trusting parent directories, inspect `.vscode` before you click Trust, and open stranger repos in a throwaway environment. If you evaluate another tool, ask how it handles an untrusted project, and harden it with the same care.

And if someone sends you a take-home test, read `tasks.json` before you read `README.md`. The README is marketing. The task file is the program.
