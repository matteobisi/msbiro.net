---
title: "Docker Agent: Declarative AI Agents You Can Version, Review, and Run Locally"
date: 2026-07-17T13:00:00Z
tags: [
  "docker", "docker-agent", "cagent", "ai-agents", "docker-model-runner",
  "docker-desktop", "local-llm", "devsecops", "cloud-native", "mcp",
  "multi-agent", "developer-tools"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Docker Agent lets you declare AI agents in YAML and run them locally on Docker Model Runner, no API key required. I installed it, ran single and multi-agent teams on my MacBook, rebuilt a real DevSecOps newsletter tool on it, and compared every feature against how I'd build the same thing by hand."
canonicalURL: "https://www.msbiro.net/posts/docker-agent-declarative-ai-agents/"
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
    alt: "Docker Agent declarative AI agents"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In my [previous post](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/) I looked at Docker Sandboxes, which answers the question of *where* to run an AI agent safely. This one is about a different piece of the same puzzle: *how* you define an agent in the first place, and whether that definition is something a team can actually own.

The tool is **Docker Agent** (`docker agent`, an open-source runtime that used to be called cagent). The pitch is that you declare an agent in YAML (its model, its instructions, its tools, and how multiple agents collaborate) and the runtime does the rest. No framework code, no glue.

I care about that claim for one specific reason. On my team, an agent that lives as a Python script in someone's home directory is a liability; an agent that lives as a reviewable YAML file in a repository is an asset. So I installed it on my MacBook (Apple Silicon, 32 GB RAM), pointed it at a local model, and ran real workloads. Everything below was executed, not summarized. The example configs and my raw test notes are in the companion repository, [POC-docker-agent](https://github.com/matteobisi/POC-docker-agent).

---

## The Problem It Solves

If you have built agents with a framework, you know the shape of the work. Wiring an LLM to a set of tools, adding a second agent that the first can delegate to, giving one of them memory, handling the tool-call loop: none of it is hard individually, but all of it is code. That code is bespoke, it drifts, and it is difficult for a colleague to review because the intent is buried in imperative plumbing.

There is also a data problem. The default path for most agent frameworks is a cloud API key, which means your prompts, your code, and sometimes your files leave the machine. For a lot of enterprise work (regulated data, customer code, anything under an NDA) that is a non-starter before the first token is generated.

Docker Agent takes a declarative approach to the first problem and leans on Docker Model Runner for the second. You describe *what* the agent is, not *how* to assemble it, and you can run the whole thing against a model that never leaves your laptop. That combination is what makes it interesting for a team rather than for a single tinkerer.

---

## Installing Docker Agent and a Local Model

The CLI ships as a Docker CLI plugin. I verified the version first:

```bash
docker agent version
# docker agent version v1.98.0
# Commit: 9cd759f9ae2bf9220d80655e9cfff9040d0aa963
```

Because I want everything local, the model comes from Docker Model Runner (DMR), which is part of Docker Desktop (enable it under **Settings > AI**). Check it is running and pull a model:

```bash
docker model status
# Docker Model Runner is running
# BACKEND    STATUS    DETAILS
# llama.cpp  Running   llama.cpp latest-metal ...

docker model pull ai/qwen3
docker model ls
```

Then confirm Docker Agent can see it:

```bash
docker agent models
# PROVIDER   MODEL             DEFAULT
# dmr        ai/qwen3:latest   *
```

That asterisk matters. I have no cloud API key configured, and Docker Agent's auto-selection falls back to a locally pulled DMR model. So a bare `docker agent run` uses a local model with zero extra configuration. No key, no `.env`, no account.

**How I'd do the same without it:** stand up a local inference server myself (llama.cpp or Ollama), expose an OpenAI-compatible endpoint, then point my framework's client at that base URL and hard-code the model name. Doable, but it is my responsibility to keep running, and it is invisible to anyone reading the agent definition. Here the model is one line of YAML (`model: dmr/ai/qwen3`) and the runtime discovers the endpoint.

---

## Running Your First Local Agent

Here is the smallest useful config, a local coding assistant with three built-in tools:

```yaml
agents:
  root:
    model: dmr/ai/qwen3
    description: A local coding assistant that reads files and runs shell commands.
    instruction: |
      You are a pragmatic software engineer working entirely on the user's
      local machine. Read files before answering questions about them, keep
      responses concise, and show the commands you would run.
    toolsets:
      - type: filesystem
      - type: shell
      - type: think
```

I validated it without spending a token first, which is handy as a pre-flight check in CI:

```bash
docker agent run --dry-run examples/01-single-agent.yaml
# Dry run mode enabled. Agent initialized but will not execute.
```

Then a real one-shot run (`--exec` skips the interactive TUI):

```bash
docker agent run --exec examples/01-single-agent.yaml \
  "List the files here and tell me in one sentence what this folder is for."
```

The agent called its `list_directory` tool, saw `.gitignore`, `NOTES.md`, and `examples/`, reasoned through the `think` tool, and answered correctly. It took about 49 seconds on the local 8B model. I ran a second task asking it to execute `uname -sm` through the shell tool; it returned `Darwin arm64` and reported macOS on Apple Silicon.

The interesting detail is in the JSON event stream (`--json`), which reports token usage per turn:

```json
{"type":"token_usage","usage":{"input_tokens":1843,"output_tokens":134,
  "last_message":{"cached_input_tokens":1804,"Cost":0,"Model":"dmr/ai/qwen3"}}}
```

`Cost: 0`. The model is local, so there is nothing to bill, and prompt caching is working even on the local backend.

**How I'd do the same without it:** a script that instantiates a model client, registers a filesystem tool and a shell tool, implements the read-eval loop that feeds tool results back to the model, and formats the output. That is the boilerplate every framework asks you to write once and then copy forever. The three lines under `toolsets:` replace it, and, more importantly, a reviewer can see at a glance that this agent can touch the filesystem and run shell commands. That visibility is a security property, not just a convenience.

---

## Tool Confirmation: Human in the Loop by Default

By default, Docker Agent prompts before running a tool call. I only used `--yolo` (auto-approve) above because `--exec` is non-interactive by nature. In day-to-day interactive use, the confirmation prompt is the right posture: the human stays in the loop for anything that touches the filesystem, the shell, or the network.

This is the same principle I wrote about with Docker Sandboxes, and the two compose. Docker Agent has a `--sandbox` flag that runs the agent inside a Docker sandbox, so you can combine the readable declarative definition with microVM-grade isolation for genuinely autonomous runs. The confirmation prompt is the checkpoint; the sandbox is the boundary.

---

## Containerized Tools via MCP

Built-in tools cover files, shell, memory, and reasoning. For anything else, Docker Agent speaks the Model Context Protocol (MCP), and the elegant part is how it runs those tools: as containers, through Docker.

```yaml
toolsets:
  - type: think
  - type: memory
    path: ./memory.db
  - type: mcp
    ref: docker:duckduckgo
```

That `ref: docker:duckduckgo` pulls and runs the DuckDuckGo MCP server as a container. I gave the agent a task that exercised both the tool and persistent memory:

```bash
docker agent run --exec examples/03-feature-showcase.yaml \
  "Use web search to find the official website of the CNCF, then store the URL in memory. Report the URL."
```

The search tool returned a real result (`https://www.cncf.io/`), and the `memory` tool wrote it to a local SQLite store:

```text
search response → Found 1 search results:
  1. Cloud Native Computing Foundation
     URL: https://www.cncf.io/
add_memory response → "Memory added successfully with ID: 1784293390988340000"
```

A local model, orchestrating a containerized third-party tool, writing to a persistent store, with no manual installation of that tool server and no API key. The memory database landed at `examples/memory.db`, resolved relative to the config file, which is a nice touch for keeping an agent's state next to its definition.

**How I'd do the same without it:** find the DuckDuckGo tool, install its runtime and dependencies on my machine (and everyone else's), keep it patched, and write the client code that connects to it. The `docker:` reference turns all of that into one line and inherits Docker's isolation for the tool process. This is the point where "Docker Agent plus Docker Desktop" stops being two products and starts being one workflow.

---

## A Multi-Agent DevSecOps Team in One YAML File

This is the feature I most wanted to test, because it maps directly onto how my team actually works: a developer writes something, and a security reviewer checks it before it ships. I described that as a three-agent team where a lead delegates to a developer and a security reviewer.

```yaml
agents:
  lead:
    model: dmr/ai/qwen3
    description: Tech lead that coordinates a developer and a security reviewer.
    instruction: |
      For any coding request: delegate implementation to the `developer`,
      send the result to the `security_reviewer`, ask for fixes if needed,
      then present the final reviewed code. Do not write code yourself.
    sub_agents:
      - developer
      - security_reviewer
    toolsets:
      - type: think

  developer:
    model: dmr/ai/qwen3
    description: Writes clean, working code.
    instruction: You are a senior developer. Write correct, minimal code.
    toolsets:
      - type: filesystem
      - type: shell
      - type: think

  security_reviewer:
    model: dmr/ai/qwen3
    description: Reviews code for security problems (a devsecops gate).
    instruction: |
      Inspect code for vulnerabilities: path traversal, injection, unsafe
      deserialization, secrets in code, missing input validation. Report
      concrete findings with severity and a suggested fix.
    toolsets:
      - type: filesystem
      - type: think
```

I gave it a task with a deliberate trap in it:

```bash
docker agent run --exec examples/02-multi-agent-team.yaml \
  "Write a Python function that reads a file whose path comes from user input and returns its text. Get it security-reviewed before you finalize."
```

The flow played out exactly as declared. The lead delegated to the developer, which wrote a `read_file_content` function with basic error handling but no path validation. The lead then passed that code to the security reviewer, which flagged the obvious problem:

> The function as written is vulnerable to path traversal attacks. The user input could include something like `../../etc/passwd`, which would let an attacker read sensitive files.

The lead came back with a hardened version that confines access to a base directory using `os.path.abspath` and `normpath`, plus a short security summary. A local model, with no orchestration code on my part, caught a real class of vulnerability because the topology said a devsecops gate had to run.

I will be honest about the cost: this took five and a half minutes on an 8B local model, because each delegation hop is a separate model turn and the small model thinks slowly. On a larger local model or a cloud model, that collapses. There is also a default `--max-iterations 20` for DMR runs, so tasks need to be scoped to finish inside that budget.

**How I'd do the same without it:** build a coordinator, define two worker agents, implement the message passing between them, decide how results flow back, and handle the loop that keeps the conversation going until the reviewer signs off. That is a real orchestration layer. Here it is `sub_agents: [developer, security_reviewer]` and three instruction blocks. And because it is YAML, the security reviewer's instructions (what it looks for, what severity language it uses) are reviewable and diffable like a policy document, which is exactly what a devsecops team wants.

---

## Distribution: Agents as Registry Artifacts

Agents can be pushed to and pulled from OCI registries, the same infrastructure as container images. I ran a pre-built agent straight from the catalog:

```bash
docker agent run --exec --model dmr/ai/qwen3 agentcatalog/pirate "Say hello in one short sentence."
# Arrr, matey! Ye be welcome to me ship!
```

That single command pulled the agent definition from the registry and ran it against my local model (`--model` overrides whatever the published agent specified). You share agents with `docker agent share push namespace/repo` and consume them with a run command, no separate distribution mechanism.

**How I'd do the same without it:** a Git repository plus a README plus a convention for how people install and run the thing, and hope everyone follows it. Reusing the OCI registry means agents get versioning, immutability, and the access controls the organisation already has for images.

---

## One Definition, Many Interfaces

The same YAML file can be served in several ways without changing the config:

- `docker agent run` for the interactive terminal UI
- `docker agent run --exec` for one-shot, scriptable use
- `docker agent serve mcp` to expose the agent itself as an MCP tool for other agents
- `docker agent serve api` for an HTTP API, and `serve a2a` for agent-to-agent protocol

That range is what lets one definition move from a developer's terminal to a shared service without a rewrite. The agent you debugged interactively is the agent you serve.

---

## A Real Use Case: A Security Newsletter, Moved Off the Cloud

I did not evaluate this in the abstract. My team runs an internal tool that builds a DevSecOps newsletter, and the reason it exists is practical: we depend on a large and growing set of tools (container runtimes, orchestrators, scanners, CI/CD components, and the rest of a modern cloud-native stack), and keeping on top of their security posture by hand does not scale. Releases, CVEs, and security-relevant changes land constantly across dozens of projects. The newsletter is how we keep that situation under control in an easy, repeatable way: it collects security-relevant updates from a set of sources (mostly GitHub release notes for projects like Kubernetes, Podman, CRI-O, and others), curates them into a structured, source-grounded draft, and publishes an approved result into Notion where the team reads it. The earlier version of that tool was driven by Claude. We rebuilt the curation layer on Docker Agent and Docker Model Runner, and it now runs on a MacBook with comparable results.

The shape of the system is worth describing, because it shows where a declarative agent fits inside a real pipeline rather than being the whole application. A deterministic Python pipeline stays in charge of the parts that must be predictable: loading sources, fetching and normalizing them, validating provenance, enforcing an approval gate, and publishing. Docker Agent owns exactly one job, curation, and it owns it declaratively.

The curator is a single versioned YAML file. It pins a local model through Docker Model Runner (no API key), sets a bounded context window for workstation-safe operation, and points at an external instruction file for the newsletter's editorial voice:

```yaml
models:
  newsletter_model:
    provider: dmr
    model: qwen3
    base_url: "${DMR_BASE_URL}"
    max_tokens: 4096
    temperature: 0.2
    provider_opts:
      context_size: 32768

agents:
  curator:
    model: newsletter_model
    description: Produces a structured, source-grounded security newsletter.
    instruction_file: instructions/newsletter-curator.md
    max_iterations: 1   # one-shot, tool-free, deterministic scheduled runs
```

Two design choices stand out to me. First, the curator is one-shot and tool-free (`max_iterations: 1`), which keeps scheduled runs deterministic; the intelligence is in the prompt and the model, not in an open-ended agentic loop. Second, publishing to Notion goes through the official Notion MCP server, run as a container next to the agent through Docker Compose, so the delivery integration is a container image rather than code we maintain.

What moving off Claude actually bought us: the prompts, the sources, and the draft never leave the machine, which matters for a security team; the cost per run went to zero; and, the point of this whole article, the curation behavior is now a reviewable YAML file plus an instruction document instead of a hosted configuration. Changing the editorial voice or swapping the model is a pull request. There is even a documented rollback (a direct Model Runner client selected by an environment variable), which is the kind of operational discipline you can only apply when the behavior is declared rather than buried.

The results are comparable to the cloud version for our purposes. A local 8B-class model is not Claude, but for summarizing release notes into a grounded newsletter with human approval before anything ships, it is good enough, and the tradeoffs land on the right side for a security team.

---

## The Integrated Picture

Individually, each of these features is useful. Together, with Docker Desktop, they form something more coherent than a single tool: Docker Model Runner supplies a local model with no key and no data egress; Docker Agent turns an agent (or a team of them) into declarative YAML; Docker Desktop runs both the model and the MCP tools as isolated containers; and Docker Sandboxes adds a microVM boundary when you want full autonomy.

For an enterprise team, that means a dev group and a devsecops group can share the exact same substrate. The developer's coding assistant and the security reviewer's gate are described in the same language, run on the same local runtime, and go through the same pull-request review. The agent definition becomes infrastructure-as-code, with everything that implies: version control, code review, reproducibility, and a clear audit trail of who changed which instruction and when. That is the property I was looking for, and it is the reason I would bring this to my team rather than a bespoke framework.

---

## Beyond Local: Cloud Models and Remote Runtimes

I tested only the local path, because local-first is the point for my use case. It is worth being clear, though, that Docker Agent is not confined to your laptop, so a team can adopt it locally and grow into a hosted deployment without changing the model.

The runtime and the model are separate concerns. The `model:` line can point at a cloud provider (OpenAI, Anthropic, Google Vertex AI, AWS Bedrock, and others) instead of a local DMR model, so you keep the same declarative agent and move only the inference off-box. The runtime itself can run as a service: `docker agent serve api` exposes agents over an HTTP API with streaming, and `docker agent run --remote <address>` connects a local invocation to a remote runtime. Agents can also reach other agents over the network through the A2A protocol (`docker agent serve a2a`), and tools can be remote too, through hosted MCP endpoints over Streamable HTTP with OAuth.

I have not verified any of that here, so treat it as documented capability rather than tested fact. The relevant point for adoption is that the same YAML that runs on my MacBook today is the artifact you would deploy to a server tomorrow. Local and remote are a deployment choice, not a rewrite.

---

## Honest Limits

Local models are the obvious tradeoff. An 8B model is slow and not as capable as a frontier cloud model, so latency is real (tens of seconds for a single agent, minutes for a delegation chain), and you should pick tasks the model can actually complete. When you need frontier quality, the previous section is the escape hatch: the same YAML runs against a cloud provider, at the cost of sending data off-box.

A couple of smaller notes from testing: `docker agent new`, the interactive config generator, needs a real terminal and fails in a piped or CI shell, so scaffold configs interactively and commit the result. And the local docs snapshot I had was older than the installed `v1.98.0` CLI, so a few documented commands (like `doctor`) did not exist; trust `docker agent --help` over any doc.

---

## Takeaways

The feature I came to evaluate, declarative multi-agent YAML, delivered on the thing I cared about: an agent stops being someone's private script and becomes a reviewable, versionable artifact that a team can own. The tool confirmation prompts, the containerized MCP tools, and the OCI distribution all reinforce that, because they keep the interesting decisions (what a tool can do, where it comes from, who can run it) visible in the definition rather than buried in code.

Running it fully local through Docker Model Runner is what makes it viable for the kind of work I actually deal with, where the data cannot leave the machine. The multi-agent run made the point concretely: a devsecops gate caught a path traversal bug not because I wrote clever code, but because the YAML said the gate had to run. That is the difference between a control that depends on discipline and one that is structural.

I would adopt this for my team, with clear eyes about the local-model latency and the experimental edges. The value is not that it makes agents possible; plenty of tools do that. The value is that it makes agents *reviewable*, and for a security-minded organisation that is the whole game. Kudos to Docker for building it in the open.

---

## Quick Reference

```bash
# Verify tooling and the local model
docker agent version
docker model status
docker agent models          # confirms a dmr/... default when no cloud key is set

# Pull a local model
docker model pull ai/qwen3

# Validate a config without spending a token
docker agent run --dry-run agent.yaml

# Run interactively (TUI)
docker agent run agent.yaml

# One-shot, non-interactive
docker agent run --exec agent.yaml "your task"

# Auto-approve tools (use inside a sandbox or for trusted tasks)
docker agent run --exec --yolo agent.yaml "your task"

# Structured event stream (token usage, tool calls)
docker agent run --exec --json agent.yaml "your task"

# Override the model at run time
docker agent run --exec --model dmr/ai/qwen3 agent.yaml "your task"

# Point the same agent at a cloud model instead (inference off-box)
docker agent run --exec --model anthropic/claude-sonnet-4-5 agent.yaml "your task"

# Run the runtime as a service, or against a remote runtime
docker agent serve api agent.yaml --listen 0.0.0.0:8080
docker agent run --remote <address> agent.yaml

# Stronger isolation for autonomous runs
docker agent run --sandbox agent.yaml

# Run a pre-built agent from the OCI registry
docker agent run agentcatalog/pirate

# Share your own agent
docker agent share push ./agent.yaml namespace/repo

# Serve one definition through different interfaces
docker agent serve mcp ./agent.yaml
docker agent serve api
```

The example configs and my full test journal are in the companion repository: [POC-docker-agent](https://github.com/matteobisi/POC-docker-agent).

---

*Tested with Docker Agent `v1.98.0`, Docker Engine `29.6.1`, and Docker Model Runner (`llama.cpp` metal backend) on a MacBook with Apple Silicon, July 2026. Local model: `ai/qwen3` (8B).*

---

## References

- [Docker Agent documentation](https://docs.docker.com/ai/docker-agent/)
- [Getting started: set up a model](https://docs.docker.com/ai/docker-agent/getting-started/set-up-a-model/)
- [Docker Model Runner](https://docs.docker.com/ai/model-runner/)
- [Multi-agent systems (delegation and handoffs)](https://docs.docker.com/ai/docker-agent/concepts/multi-agent/)
- [API server and remote runtime](https://docs.docker.com/ai/docker-agent/features/api-server/)
- [Remote MCP servers](https://docs.docker.com/ai/docker-agent/features/remote-mcp/)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [Docker Agent Catalog on Docker Hub](https://hub.docker.com/u/agentcatalog)
- [My previous post: Docker Sandboxes](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/)
- [Companion repository: POC-docker-agent](https://github.com/matteobisi/POC-docker-agent)
