---
title: "Docker Agent: Turning AI Agents Into a Shared Team Capability"
date: 2026-07-22T10:26:00Z
tags: [
  "docker", "docker-agent", "cagent", "ai-agents", "docker-model-runner",
  "docker-desktop", "local-llm", "devsecops", "cloud-native", "mcp",
  "engineering-management", "ai-governance"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A DevSecOps team leader's evaluation of Docker Agent: how declarative YAML, OCI distribution, and local-model support turn AI agents into a standardised, shareable, low-skill engineering capability rather than a personal experiment."
canonicalURL: "https://www.msbiro.net/posts/docker-agent-shareable-ai-agents/"
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
    alt: "Docker Agent: a shareable AI agent capability for engineering teams"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

As I move further into engineering management, I evaluate new technology through a different lens. I am not only asking whether a tool is interesting or whether I can build a convincing proof of concept. I am asking whether it can help an engineering team work more consistently, securely, and efficiently over time.

AI agents are now part of that discussion. They can support research, documentation, code review, operational triage, and repetitive internal workflows. The management question is not simply *"Can we build an agent?"* It is: **how do we make agents understandable, shareable, and affordable for a team, without requiring every contributor to become an agent-framework expert?**

I explored [Docker Agent](https://docs.docker.com/ai/docker-agent/) with this question in mind, using my companion [POC-docker-agent](https://github.com/matteobisi/POC-docker-agent) repository, its [example agent definitions](https://github.com/matteobisi/POC-docker-agent/tree/main/examples), and its [test journal](https://github.com/matteobisi/POC-docker-agent/blob/main/NOTES.md). This is not a tutorial or an argument for automating everything. It is an evaluation of one possible approach for turning AI-agent experiments into a manageable engineering capability that can be passed around the team.

---

## The Team Need Is Agent Management, Not Another Demo

Most teams start their AI-agent journey in a familiar way. An engineer has a useful idea, writes a Python application around an LLM API, connects a tool, and produces something valuable. That is a good way to learn.

The challenge appears once the experiment becomes useful enough for other people to depend on it. A second engineer needs to run it. Someone changes a prompt. The agent gains access to a filesystem, shell, ticketing platform, or internal API. A cloud-model bill grows unexpectedly. A security reviewer needs to understand what the agent is authorised to do.

At that point, the team is no longer managing a small script; it is managing a system. From a leadership perspective, I want clear answers to a few questions:

| Question | Why it matters to a team |
| --- | --- |
| What is this agent responsible for? | It keeps the scope understandable and measurable. |
| Which model does it use? | It makes quality, cost, and data handling a deliberate choice. |
| Which tools can it call? | It makes permissions visible before the agent runs. |
| Which other agents can it delegate to? | It makes workflow controls explicit rather than implicit in code. |
| Where does the definition come from? | It enables review, versioning, rollback, and reproduction. |

These are not theoretical governance questions. They are practical foundations for reliable engineering work, and they double as the criteria I use to decide whether a skill is genuinely *shareable* across a team or quietly dependent on the person who wrote it.

---

## The Cost of "Just Python"

Python remains an excellent choice for AI development. It is flexible, mature, and familiar to many engineers. For genuinely custom workflows, bespoke application code is often the right answer.

However, every agent built from scratch also becomes another application for the team to operate. Someone must maintain dependencies, model-provider integrations, tool schemas, orchestration logic, retries, state management, configuration, secrets, packaging, and documentation. Frameworks like LangChain, CrewAI, and LlamaIndex speed up the first build but do not remove that operational surface; they shift it from hand-rolled code to framework-specific idioms that still need a maintainer.

That effort is justified when the agent itself is a differentiating product. But many internal use cases are simpler: a defined role, a controlled set of tools, a repeatable workflow, and human approval at the right point.

The risk is not Python, and it is not the frameworks. The risk is allowing useful internal automation to become dependent on one person's laptop, local environment, undocumented prompt choices, or personal API key. For a DevSecOps team, that is unnecessary operational complexity and an avoidable bus-factor risk.

We already use versioned, declarative artefacts to manage infrastructure, security policy, CI/CD pipelines, and Kubernetes workloads. It is reasonable to expect similar visibility for AI-agent behaviour.

---

## What Docker Agent Offers

Docker Agent is an open-source framework for defining and running specialised AI agents. Its source code is published on GitHub under the [Apache License 2.0](https://github.com/docker/docker-agent/blob/main/LICENSE). It is available through Docker Desktop and through standalone installation paths, including Homebrew, Winget, GitHub releases, and source builds.

Its key idea is straightforward: define an agent in YAML rather than embedding every important decision in custom orchestration code. An agent definition can make its role, instructions, model, tools, sub-agents, and operational settings visible. Compared with a code-first framework, the skill floor for *reviewing and running* an agent drops sharply: a reviewer does not need to understand a provider SDK, a callback graph, or a prompt template engine to see what an agent is allowed to do.

This does not remove the need for engineering judgement. Instead, it makes that judgement easier to inspect. A pull request can show that someone changed an agent's model, expanded filesystem or network access, added an MCP integration, modified security-review instructions, or introduced a new delegation path. That is a much more productive review conversation than asking a reviewer to infer permissions and behaviour from a large amount of framework-specific code.

To make this concrete, here is a reduced version of the multi-agent team from my POC. A lead agent coordinates a developer and a security reviewer, all on a local model:

```yaml
agents:
  lead:
    model: dmr/ai/qwen3
    description: Tech lead that coordinates a developer and a security reviewer.
    instruction: |
      You are a pragmatic tech lead. For any coding request:
        1. Delegate the implementation to the `developer`.
        2. Send the developer's code to the `security_reviewer`.
        3. If the reviewer finds issues, ask the developer to fix them.
        4. Present the final, reviewed code and a one-line security summary.
      Do not write code yourself; coordinate the specialists.
    sub_agents:
      - developer
      - security_reviewer
    toolsets:
      - type: think

  developer:
    model: dmr/ai/qwen3
    description: Writes clean, working code.
    instruction: |
      You are a senior developer. Write correct, minimal code for the task.
      When asked to fix review findings, apply the fix and explain what changed.
    toolsets:
      - type: filesystem
      - type: shell
      - type: think

  security_reviewer:
    model: dmr/ai/qwen3
    description: Reviews code for security problems (a devsecops gate).
    instruction: |
      You are a security reviewer. Inspect code for vulnerabilities such as
      path traversal, injection, unsafe deserialization, secrets in code, and
      missing input validation. Report concrete findings with severity and a
      suggested fix. If the code is safe, say so explicitly.
    toolsets:
      - type: filesystem
      - type: think
```

The full, runnable definitions, including a single-agent starter and a feature showcase with an MCP web-search tool, are in the [examples folder of the POC repository](https://github.com/matteobisi/POC-docker-agent/tree/main/examples). The point of showing the snippet here is not the syntax; it is who can now participate. A new joiner, an ops engineer, or a security reviewer can read that file, understand the agent's role and permissions in a few minutes, and propose a change with a one-line diff. That is the standardisation effect I was looking for.

---

## Local and Remote by Design

One capability I find particularly useful is that the agent definition can be separated from the model-execution decision. Docker Agent can work with local models through [Docker Model Runner](https://docs.docker.com/ai/model-runner/) and can also use remote providers, including Anthropic, OpenAI, Google, and AWS Bedrock.

The same agent workflow can therefore support different cost, performance, and data-handling requirements.

| Model approach | Best suited to | Main consideration |
| --- | --- | --- |
| Local model | Sensitive internal material, routine tasks, high-volume drafting, predictable usage | Hardware capacity, latency, and model quality |
| Remote model | Difficult reasoning tasks and work where high capability has clear value | API cost and external data boundary |
| Mixed approach | Teams with varied workloads and risk profiles | Requires clear rules for model selection |

In my test environment, I used a local Qwen 8B model on an Apple Silicon MacBook without cloud credentials. The proof of concept demonstrated filesystem and shell tools, a small development-and-security-review workflow, and a containerised MCP tool.

This matters because teams should not have to redesign an entire workflow when the model decision changes. A manager should be able to ask: *is this task sensitive, frequent, expensive, or quality-critical?* The team can then choose a suitable model path without discarding the agent definition, which is exactly the kind of decoupling that keeps a shared skill durable as vendors and models shift.

---

## A Better Delivery Model: Standardisation as Knowledge Transfer

The most compelling part of the Docker approach, for me, is not only YAML. It is distribution.

Docker Agent can package and distribute agent definitions as OCI artefacts, using the same registry model that teams already use for container images. This makes an agent easier to version, retrieve, promote, and roll back through familiar engineering practices. An approved agent definition can be shared through a controlled registry, developers can reproduce the same agent version on different workstations, teams can tag known-good versions and roll back when a change degrades outcomes, a shared service can retrieve an approved definition rather than relying on manual local setup, and the model choice can remain local or environment-specific while the workflow definition stays consistent.

This is where an agent stops being a personal productivity tool and becomes a shared team asset, and it is also where the "shareable skill" idea becomes concrete. The same definition that a senior engineer authors can be handed to another squad, a new hire, or an on-call engineer who runs the identical thing without re-deriving the prompt, the tool wiring, or the delegation rules. The knowledge travels with the artefact rather than living in one person's head.

For a manager, reproducibility is not bureaucracy. It is what allows a team to improve safely. If a prompt update, new tool, or model change causes weaker output, we should be able to identify what changed, review the difference, and restore a known working version.

---

## Security Is a Workflow Property

AI agents should not replace human review in security-sensitive work. They should help teams make good practices easier to repeat.

In my proof of concept, a lead agent delegated an implementation task to a developer agent and then passed the result to a security-review agent, the team shown in the snippet above. When the implementation did not properly constrain a user-supplied file path, the security reviewer identified the path-traversal risk and suggested a remediation before the final response was produced.

The important lesson is not that an AI agent can replace a security engineer. It cannot. The lesson is that, when a team has decided a security pass is needed for a class of work, that decision can be made part of the defined workflow rather than an informal intention.

Human judgement, scoped credentials, logging, approvals, and normal security controls remain essential. Docker Agent can make the agent-specific controls more explicit and easier to review, which is why I frame the benefit as visibility and reproducibility rather than full governance: governance also depends on logging, audit, and spend controls that the tool does not solve on its own.

The best early use cases are bounded and reviewable. The work I would start with is creating first drafts of internal documentation, summarising validated technical or security information, classifying repetitive requests, preparing structured material for human review, and supporting recurring development or operational workflows. In each case quality can be measured, the scope is clear, and a person retains responsibility for the outcome. I would not begin with unrestricted autonomy.

---

## Efficiency Includes Cost Discipline

Local inference is attractive for work that is frequent, sensitive, or predictable. Once a local model has been downloaded, there is no per-token provider charge, and prompts can remain on the machine. However, local inference is not cost-free: hardware, electricity, support, storage, and latency still matter.

My multi-agent test took approximately five and a half minutes on an 8B local model. That is reasonable for scheduled or asynchronous work, but not necessarily for an interactive developer experience.

This is exactly why management matters. The goal is not to declare that local models are always cheaper or remote models are always better. The goal is to match technology to the workload. Use local models when privacy and predictable marginal costs are priorities; use remote models when stronger performance creates enough business value to justify the cost and data boundary; avoid multi-agent designs where a simple deterministic workflow would be more reliable; and measure quality, latency, and cost before scaling usage.

A declarative model helps because these choices can evolve without rebuilding the workflow from scratch.

---

## Caveats and Adoption Risks

A balanced evaluation has to name what Docker Agent does not solve. The framework is young, its API surface and YAML schema can still change, and relying on it ties the team to an emerging tool that may shift under us. Declarative YAML improves visibility but does not, by itself, provide agent evaluation quality, tracing, observability, or spend limits; those still need to be layered on, whether through logging, an evaluation harness, or platform controls around the registry and the model endpoints. There is also an adoption cost: the team has to learn a small new vocabulary of agents, toolsets, and MCP refs, and someone has to own the registry hygiene and promotion rules.

None of this is a blocker. It is simply the reminder that standardisation is not the same as maturity, and that a shareable skill still needs an owner who keeps the catalogue coherent.

---

## What I Take From This

I like Docker Agent because it offers a practical path to treating AI agents as controlled, shareable engineering assets rather than individual experiments.

Its declarative definitions make behaviour easier to inspect and lower the skill floor for reviewing and running an agent. Version control and OCI-based distribution make agents easier to reproduce, hand off, and roll back, turning knowledge transfer into a registry operation rather than a README hunt. Local-model support offers a privacy-conscious option with predictable usage costs, while remote-model support leaves room for tasks that need greater capability.

As a DevSecOps team leader, this is the value I see in studying emerging technology. It is not about adopting every new tool or replacing engineers with automation. It is about identifying technologies that help the team reduce accidental complexity, make better security and cost decisions, and spend more time on valuable work.

Docker Agent will not eliminate the need for architecture, engineering discipline, governance, or human accountability. But it can provide a useful foundation for managing internal AI agents in a way that is more visible, reproducible, and sustainable, and that more of the team can operate without first becoming framework specialists.

That is the standard I want to apply to AI adoption: not impressive demos, but capabilities that help a team work better together.

---

## References

- [Docker Agent documentation](https://docs.docker.com/ai/docker-agent/)
- [Docker Agent source code and Apache License 2.0](https://github.com/docker/docker-agent)
- [Docker Model Runner](https://docs.docker.com/ai/model-runner/)
- [Setting up local and cloud models](https://docs.docker.com/ai/docker-agent/getting-started/set-up-a-model/)
- [Docker Agent API server](https://docs.docker.com/ai/docker-agent/features/api-server/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Companion POC, example agent definitions, and verified test journal](https://github.com/matteobisi/POC-docker-agent)

*Tested locally with Docker Agent `v1.98.0`, Docker Engine `29.6.1`, and Docker Model Runner on a MacBook with Apple Silicon in July 2026. The POC used an 8B Qwen model. Product capabilities and packaging can change, so consult the linked Docker documentation before adopting it in a production environment.*