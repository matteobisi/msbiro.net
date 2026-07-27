---
title: "Shadow AI in CI/CD: Threat-Modelling Laptop to Kubernetes"
date: 2026-07-27
tags: [
  "shadow-ai", "ai-security", "ai-agents", "mcp", "prompt-injection",
  "cicd-security", "supply-chain-security", "kubernetes",
  "devsecops", "cloud-native", "security"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Shadow AI turns ungoverned coding assistants and agents into a live threat across CI/CD. A stage-by-stage threat model from developer laptop to Kubernetes pod, with the executive controls and tooling that contain the risk."
canonicalURL: "https://www.msbiro.net/posts/shadow-ai-cicd-kubernetes-threat-model/"
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
    alt: "Shadow AI threat model across a CI/CD pipeline from developer laptop to a Kubernetes pod"
    caption: "Mapping where ungoverned AI tools and agents create risk along the software delivery path"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Artificial intelligence is becoming part of daily software delivery, often before it becomes part of the organisation's security architecture. This creates **Shadow AI**: AI tools, models, agents, extensions, or integrations used without formal approval, ownership, risk assessment, or monitoring.

For executives, Shadow AI is not primarily a "developers using ChatGPT" issue. It is an enterprise-risk issue: ungoverned AI can gain access to source code, intellectual property, credentials, customer data, cloud environments, and deployment workflows. When AI systems are allowed to call tools or take actions, they must be treated as new non-human identities with access rights, not simply as productivity software.

This article threat-models a typical CI/CD journey: from a developer laptop to a workload running in a Kubernetes pod.

## What Shadow AI Means

Shadow AI includes any AI capability adopted outside an approved governance process. It may be a browser-based coding assistant, a VS Code extension, an unofficial model API key in a repository, an AI code-review bot connected to GitHub, or an agent able to open tickets, change infrastructure, trigger pipelines, or access Kubernetes.

The risk rises sharply when AI moves from **advisory** use to **action** use. An assistant that suggests a code snippet creates one type of risk; an agent with a GitHub token, cloud credentials, or Kubernetes permissions can create, alter, or delete production resources at machine speed.

The business problem is therefore visibility and accountability:

- Which AI tools and agents are in use?
- What data do they receive?
- Which systems can they access?
- Who is accountable for each agent's behaviour?
- Can access be revoked immediately if an agent behaves unexpectedly?

An effective governance model assigns every agent a human owner, registers it as an identifiable workload, constrains it through least privilege, and continuously monitors its actions.

## The Delivery Path

Consider a common modern delivery path, from developer laptop through source control, CI, the artifact registry, and CD, to a Kubernetes pod:

[![Shadow AI threat model across the CI/CD delivery path: at each stage an ungoverned AI tool or agent (left) is paired with the defensive control that clamps it (right), from developer laptop to Kubernetes pod](delivery-path-shadow-ai.png)](delivery-path-shadow-ai.png)

*Click the diagram to open it full size.*

Shadow AI can appear at every stage. The important point for leaders is that a small convenience decision at the beginning of the path can become a production-security exposure at the end.

| Delivery stage | Typical Shadow AI use | Primary business risk |
| :-- | :-- | :-- |
| Developer laptop | Unapproved code assistant, local AI plugin, public chatbot | Source code, secrets, architecture, or customer information may leave approved boundaries |
| Source control | AI bot reviews PRs, generates commits, summarises repositories | Excessive repository permissions, malicious or unsafe code changes, lack of ownership |
| CI pipeline | AI generates pipeline logic, analyses logs, creates infrastructure changes | Build secrets or cloud credentials may be exposed; automated changes can compromise the supply chain |
| Artifact registry | AI-assisted image creation or dependency selection | Vulnerable, malicious, or untraceable dependencies may reach production |
| CD platform | AI agent approves, modifies, or rolls out releases | Unauthorised deployment, bypassed change controls, poor traceability |
| Kubernetes runtime | Agent queries clusters, remediates alerts, scales workloads | Over-privileged service accounts, destructive actions, lateral movement, data exposure |

## Threat Model

A threat model does not require executives to predict every attack. It requires the organisation to identify valuable assets, likely abuse paths, and the controls that limit impact.

### Assets to Protect

In this scenario, the highest-value assets include:

- Source code and proprietary algorithms
- API keys, tokens, certificates, and other secrets
- Customer, employee, and commercial data
- CI/CD configuration and software-signing keys
- Cloud and Kubernetes identities
- Container images and software supply-chain integrity
- Production availability and the organisation's reputation

### Threat Actors

Shadow AI does not necessarily begin with a malicious employee. Often, a well-intentioned engineer adopts a tool to move faster. Threat actors then exploit the resulting blind spot.

Relevant actors include:

- External attackers targeting exposed AI integrations or stolen credentials
- Malicious insiders abusing poorly governed access
- Third parties or compromised AI-tool providers
- Attackers using prompt injection to manipulate an agent
- A legitimate AI agent acting incorrectly because of ambiguous instructions, unsafe context, or excessive permissions

AI-agent security guidance, such as the [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html), increasingly focuses on prompt injection, sensitive-data exposure, unsafe tool use, and actions that fall outside approved user intent.

## Attack Paths by Stage

### 1. Developer Laptop

A developer installs an AI coding extension or pastes an error log into a public AI service. That log contains an API token, internal hostname, customer identifier, or part of a production configuration.

The immediate issue may appear minor, but the organisation has lost visibility over where proprietary data was sent, how it may be retained, and whether it is used for model training or third-party processing. A second risk emerges when the assistant suggests a dependency, configuration, or infrastructure command that the developer trusts without validation.

**Management control:** provide approved AI tools that developers can use safely. A blanket ban normally pushes usage further into the shadows; a governed alternative creates visibility, training, and enforceable data-handling rules. Where an agent genuinely needs broad local permissions, [running it inside an isolated microVM sandbox](/posts/docker-sandboxes-ai-agents/) keeps that autonomy away from the host filesystem, SSH keys, and cloud credentials.

### 2. Source Control

An unapproved AI bot is connected to GitHub, GitLab, or Bitbucket with broad permissions. It can read all repositories, comment on pull requests, create branches, or write code.

The threat is not only code leakage. If the integration token is stolen, or an attacker manipulates the bot through a malicious issue, pull-request description, or documentation file, the bot may create unsafe code changes or disclose repository content.

**Management control:** every AI integration needs a named business owner, separate identity, minimal repository scope, short-lived credentials where possible, and auditable logs. An agent should not receive organisation-wide repository access merely because it performs code review for one team.

### 3. CI Pipeline

CI/CD systems hold some of the organisation's most powerful credentials: source-control tokens, package-registry credentials, cloud access keys, signing keys, and deployment secrets.

A Shadow AI capability in the pipeline may inspect build logs, generate scripts, or autonomously "fix" a failing build. The danger is that it becomes an ungoverned privileged operator. A prompt-injection payload hidden in source code, a README file, a dependency description, or a build log could influence the agent's interpretation and cause it to reveal data or execute an unsafe action.

**Management control:** prohibit long-lived secrets in prompts, logs, and build environments. AI-connected pipeline jobs should run in isolated environments, use tightly scoped ephemeral credentials, and require policy checks and human approval before they can alter infrastructure or release software. The [sandbox isolation model](/posts/docker-sandboxes-ai-agents/) that contains an agent on a developer machine applies equally to a build runner.

### 4. Artifact Registry and Supply Chain

AI-generated code can introduce insecure packages, weak configurations, or dependencies that were never properly reviewed. An AI tool may also recommend a container base image or copied snippet without checking provenance, licensing, vulnerabilities, or maintenance status.

For leadership, this is a software-supply-chain issue: if teams cannot establish what went into an image, who approved it, and whether it passed security gates, then rapid delivery becomes unmanaged operational risk.

**Management control:** require image scanning, software-bill-of-material generation, signed artifacts, dependency policies, and promotion gates. AI-generated code should follow the same review, testing, and release requirements as human-written code.

### 5. Continuous Delivery

An AI release agent may be able to modify Helm charts, update GitOps manifests, change deployment targets, approve releases, or trigger rollback actions. If that agent is connected to production without clear boundaries, it can bypass the governance controls that executives expect from a mature change-management process.

**Management control:** distinguish between an agent that *recommends* a release action and one that *executes* it. High-impact actions such as production deployment, privilege escalation, data export, deletion, or network-policy changes should have explicit approval gates and strong auditability.

[Okta's guidance for AI agents](https://www.okta.com/solutions/secure-ai/) recommends verified user context, granular permissions, lifecycle management, real-time monitoring, and pausing high-stakes workflows for human approval.

### 6. Kubernetes Runtime

The final stage is often the most critical. A Shadow AI remediation agent might be given `cluster-admin` permissions "temporarily" to investigate an alert or fix failed workloads. In practice, this turns a convenience tool into a high-value target.

A compromised or manipulated agent with broad Kubernetes permissions could enumerate secrets, deploy a malicious workload, alter network policies, exfiltrate data, or disrupt production. Kubernetes does not distinguish between a harmful action performed by an attacker and one performed by an over-privileged automation identity.

**Management control:** apply namespace boundaries, workload identity, minimal RBAC, admission controls, runtime detection, and network segmentation. An AI agent responsible for restarting a deployment in one namespace should not be able to read secrets or manage cluster-wide resources.

## Executive Controls That Matter

The goal is not to stop AI adoption. It is to make AI use visible, accountable, and proportionate to its risk.

### 1. Build an AI Inventory

Create a living inventory of approved and discovered AI tools, models, browser extensions, code assistants, AI agents, API integrations, and MCP servers.

For each entry, record:

- Business owner and technical owner
- Purpose and users
- Data classification allowed
- Connected systems and permissions
- Model or provider used
- Production status
- Security review date
- Incident-response and access-revocation process

Agent discovery, inventory, risk assessment, tool-access control, and runtime action control are key capabilities promoted by enterprise AI-security platforms. Open-source registries for agents and MCP tools, such as agentregistry, aim at the same problem and can scan connected runtimes to reveal what was never registered in the first place.

### 2. Treat AI Agents as Identities

Every agent should have a unique identity. It should never borrow a developer's personal credentials or use a shared administrator account.

Use least privilege by default:

- Separate credentials per environment
- Short-lived tokens rather than persistent secrets
- Read-only access where possible
- Namespace-level Kubernetes permissions
- Explicit allowlists for APIs, tools, repositories, and MCP servers
- Immediate revocation capability

### 3. Separate Advice from Execution

Not every AI function needs the same level of control.

| AI capability | Recommended control level |
| :-- | :-- |
| Code explanation or documentation drafting | Approved tool, data classification rules, developer training |
| Code suggestion | Human review, standard testing, secret scanning, dependency checks |
| Pull-request creation | Restricted repository permissions, mandatory peer review |
| CI/CD pipeline modification | Isolated execution, policy-as-code checks, approval gate |
| Cloud or Kubernetes remediation | Strict RBAC, limited scope, full audit trail, human approval for high-impact actions |
| Autonomous production changes | Exceptional approval only, time-bound access, kill switch, continuous monitoring |

### 4. Apply Defence in Depth

Prompt filtering alone is not enough. Prompt injection is a meaningful risk, but it is only one control layer.

A mature architecture combines:

- AI and data governance
- Identity and access management
- Source-control protections
- Secrets management
- Secure CI/CD configuration
- Dependency and container scanning
- Kubernetes admission and runtime policies
- Network controls
- Centralised logging and incident response

The principle is simple: if one control fails, the agent must still be unable to cause material harm.

## Tooling Options

No vendor solves the entire Shadow AI problem. Leaders should select tools according to the risk area they need to improve, then integrate them into a coherent security architecture.

| Area of interest | Commercial options | Free or open-source options | What it helps address |
| :-- | :-- | :-- | :-- |
| AI discovery, governance, prompt and runtime protection | SentinelOne Prompt Security, Tigera Lynx, Palo Alto Networks, Check Point | agentgateway, agentregistry, Snyk Agent Scan, OWASP guidance | Discover AI use, inspect prompts and responses, reduce sensitive-data leakage, detect unsafe agent actions |
| AI and application security | Snyk | Semgrep Community Edition, OWASP Dependency-Check | Secure AI-generated and conventional application code, dependencies, containers, and infrastructure-as-code |
| Identity for AI agents | Okta/Auth0, cloud-provider IAM offerings, Tigera Lynx | Keycloak, SPIFFE/SPIRE | Establish agent identity, enforce least privilege, maintain accountability, revoke access |
| CI/CD and software supply-chain security | GitHub Advanced Security, GitLab Ultimate, JFrog, Snyk | Trivy, Syft, Grype, Sigstore/Cosign | Scan code and images, generate SBOMs, sign artifacts, prevent risky releases |
| Kubernetes policy and runtime protection | Tigera/Calico, Wiz, Palo Alto Prisma Cloud, Sysdig | Kyverno, OPA Gatekeeper, Falco, Cilium, Kubescape | Enforce deployment policy, reduce cluster permissions, segment networks, detect suspicious runtime behaviour |
| Secrets management | HashiCorp Vault, cloud-native secret managers | External Secrets Operator, Sealed Secrets, SOPS | Prevent tokens and credentials from entering repositories, prompts, logs, and container images |

SentinelOne's [Prompt Security](https://prompt.security/solutions/homegrown-genai-apps) capability is relevant to AI-layer governance and protection, while Snyk is most relevant to developer, application, dependency, container, and infrastructure-as-code security. Tigera covers two distinct areas: the Calico platform for Kubernetes networking, policy enforcement, and workload protection, and Lynx for agent-specific control inside a cluster. Lynx discovers the agents running in an estate, issues each one a cryptographic identity, scopes credentials to a single hop, and evaluates LLM, MCP, and tool calls against default-deny policy, using kernel-level instrumentation to flag an agent behaving abnormally even when it holds a valid credential. It integrates with existing identity providers rather than replacing them. These should be assessed as complementary control areas rather than interchangeable products.

Open-source controls are a credible starting point for technically mature teams, especially Kyverno or OPA Gatekeeper for Kubernetes admission policies, Falco for runtime detection, Cilium for network segmentation between namespaces and workloads, SPIFFE/SPIRE for workload identity, Trivy for image and configuration scanning, and Sigstore for artifact signing.

At the agent layer specifically, a newer set of Apache 2.0 projects has emerged to close the gap between having a policy and being able to enforce it on agent traffic:

- **[agentgateway](https://agentgateway.dev/)** is a proxy for MCP and Agent2Agent (A2A) traffic, contributed by Solo.io to the Linux Foundation. It supports JWT, API-key and OAuth authentication, role-based authorisation through a CEL policy engine, MCP federation across multiple tool servers, content filtering, rate limiting, and OpenTelemetry tracing. It gives an organisation a single enforcement point between agents, the tools they call, and the models behind them.
- **[agentregistry](https://github.com/agentregistry-dev)** is a registry for MCP servers, agents, and agent skills, contributed by Solo.io and submitted to the CNCF Sandbox in March 2026. The application is still under review, so treat it as an early-stage project rather than a CNCF-governed one. Alongside curated discovery through a UI, CLI, and API, it can scan connected runtimes to surface inventory that was never formally registered, which is directly useful for the discovery problem described earlier.
- **[Snyk Agent Scan](https://github.com/snyk/agent-scan)**, formerly MCP-Scan from Invariant Labs, scans MCP servers, agent skills, and the agent configuration files used by common developer tools. It covers more than fifteen risk categories, including prompt injection, tool poisoning, and data-leakage flows across connected components. It is Apache 2.0 but developed by Snyk without external contributions.

These three are young and should be evaluated as such, but their relevance is structural rather than promotional: an agent gateway is the natural chokepoint for the least-privilege and allowlisting controls recommended above, a registry is the natural home for the AI inventory, and an MCP scanner covers a supply-chain surface that conventional dependency scanners do not inspect.

Open source does not eliminate the need for operational ownership. Someone must maintain policies, triage findings, manage integrations, and prove control effectiveness, whether the tool is a Kubernetes admission controller or an agent gateway.

## Conclusion

Shadow AI is the next evolution of Shadow IT, but with a critical difference: modern AI agents can interpret information, call tools, and act across engineering and business systems. That makes them powerful accelerators. It also makes them potential high-speed pathways to source-code leakage, credential compromise, supply-chain abuse, and production disruption.

The executive decision is not whether developers will use AI. They already are. The decision is whether the organisation will manage AI as an untracked collection of productivity tools or as a governed ecosystem of identities, data flows, and production-capable services.

A secure path forward is clear: discover AI use, assign ownership, restrict permissions, protect the software supply chain, enforce Kubernetes boundaries, monitor agent behaviour, and retain human approval for consequential actions. AI agents should be allowed to help teams move faster, but never allowed to operate beyond the organisation's ability to see, control, and stop them.