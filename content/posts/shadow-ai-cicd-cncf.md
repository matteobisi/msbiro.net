---
title: "Shadow AI in CI/CD: Threat-Modeling the Path from Developer Laptop to Kubernetes"
date: 2026-07-23T09:00:00Z
tags: [
  "shadow-ai", "ai-agents", "supply-chain-security", "cicd", "kubernetes",
  "cloud-native", "devsecops", "cncf", "policy-as-code", "workload-identity"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A practical threat model of the CI/CD path from developer laptop to Kubernetes pod against Shadow AI, mapped to CNCF and open source security controls."
canonicalURL: "https://www.cncf.io/blog/"
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
    image: ""
    alt: "Delivery path from developer laptop to Kubernetes pod with Shadow AI injection points"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Artificial intelligence is becoming part of daily software delivery, often before it becomes part of the security architecture. That gap has a name: **Shadow AI**. It is any AI tool, model, agent, extension, or integration used in the software lifecycle without formal approval, ownership, risk assessment, or monitoring.

For platform and security teams, Shadow AI is not really a "developers using a chatbot" problem. It is an access problem. Ungoverned AI can reach source code, secrets, customer data, cloud environments, and deployment workflows. And once an AI system is allowed to *call tools* and *take actions*, it stops being productivity software and becomes a new non-human identity with permissions, a blast radius, and a place in your threat model.

This article threat-models a common cloud-native delivery path, from a developer laptop to a workload running in a Kubernetes pod, and maps each stage to controls you can implement today with CNCF and open source projects.

## From advisory to action

The risk rises sharply when AI moves from **advisory** use to **action** use.

An assistant that suggests a code snippet creates one class of risk: data leaving an approved boundary, or a subtly wrong suggestion trusted without review. An agent with a Git token, cloud credentials, or a Kubernetes ServiceAccount creates a different class entirely. It can create, alter, or delete resources at machine speed, and Kubernetes will not distinguish between a harmful action taken by an attacker and the same action taken by an over-privileged automation identity.

So the questions worth answering are operational, not philosophical:

- Which AI tools and agents are in use?
- What data do they receive?
- Which systems can they reach, and with what permissions?
- Who owns each agent's behavior?
- Can access be revoked immediately if an agent misbehaves?

A workable model gives every agent a human owner, registers it as an identifiable workload, constrains it with least privilege, and monitors what it actually does.

## The delivery path

Consider a typical cloud-native delivery path:

[![Shadow AI threat model across the CI/CD delivery path: at each stage an ungoverned AI tool or agent (left) is paired with the CNCF/FOSS control that clamps it (right), from developer laptop to Kubernetes pod](delivery-path-shadow-ai.png)](delivery-path-shadow-ai.png)

<small>Click the diagram to open it full size.</small>

Shadow AI can appear at every stage, and a small convenience decision at the start of the path can become a production exposure at the end.

| Delivery stage | Typical Shadow AI use | Primary risk |
| :-- | :-- | :-- |
| Developer laptop | Unapproved code assistant, local model plugin, public chatbot | Source code, secrets, or architecture leave approved boundaries |
| Source control | AI bot reviews PRs, generates commits, summarizes repositories | Excessive repo permissions, unsafe code changes, no clear ownership |
| CI pipeline | AI generates pipeline logic, analyzes logs, "fixes" failing builds | Build secrets and cloud credentials exposed; automated supply-chain changes |
| Artifact registry | AI-assisted image or dependency selection | Vulnerable, malicious, or untraceable dependencies reach production |
| CD platform | AI agent approves, modifies, or rolls out releases | Bypassed change controls, unauthorized deployment, poor traceability |
| Kubernetes runtime | Agent queries clusters, remediates alerts, scales workloads | Over-privileged ServiceAccounts, destructive actions, lateral movement |

## Threat model

A threat model does not require you to predict every attack. It requires you to name the valuable assets, the likely abuse paths, and the controls that limit impact.

### Assets to protect

- Source code and proprietary algorithms
- API keys, tokens, certificates, and other secrets
- Customer, employee, and commercial data
- CI/CD configuration and software-signing keys
- Cloud and Kubernetes identities
- Container images and software supply-chain integrity
- Production availability and reputation

### Threat actors

Shadow AI rarely starts with a malicious insider. More often a well-intentioned engineer adopts a tool to move faster, and attackers exploit the resulting blind spot. Relevant actors include:

- External attackers targeting exposed AI integrations or stolen credentials
- Malicious insiders abusing poorly governed access
- Compromised third-party or AI-tool providers
- Attackers using **prompt injection** to manipulate an agent
- A legitimate agent acting incorrectly because of ambiguous instructions, unsafe context, or excessive permissions

Prompt injection is the through-line. Agents routinely read untrusted content: issue descriptions, READMEs, dependency changelogs, build logs. Any of it can steer an agent into disclosing data or taking an unsafe action, which is why prompt filtering alone will never be enough and defense-in-depth is the real answer. The OWASP guidance on AI agents and LLM applications is a good baseline for these failure modes.

## Attack paths by stage, and the controls that clamp them

### Developer laptop

A developer installs an AI coding extension or pastes an error log into a public service. That log contains an API token, an internal hostname, or a customer identifier. The organization has now lost visibility over where proprietary data went and how it may be retained. A second risk follows: the assistant suggests a dependency or command that the developer trusts without validation.

**Defensive control.** Provide *approved* AI tools so usage is visible instead of hidden; a blanket ban tends to push it further into the shadows. Back this with pre-commit secret scanning so tokens never reach a shared surface in the first place, and enforce signed commits. **Gitsign** (part of the Sigstore project) lets developers sign commits with short-lived, identity-based certificates instead of long-lived GPG keys, which makes "who produced this change" auditable, including when the author is an agent.

### Source control

An unapproved AI bot is connected to your Git host with broad permissions. It can read every repository, comment on pull requests, create branches, or push code. The threat is not only leakage: if the integration token is stolen, or the bot is manipulated through a malicious issue or PR description, it can introduce unsafe changes or disclose repository content.

**Defensive control.** Give every AI integration a named owner, its own identity, minimal repository scope, and short-lived credentials. An agent that reviews code for one team should not hold org-wide repository access. Require review gates and provenance on everything merged, and treat agent-authored PRs like any other untrusted contributor: mandatory human review, no self-approval.

### CI pipeline

CI systems hold some of the most powerful credentials you own: source-control tokens, registry credentials, cloud keys, and signing keys. A Shadow AI capability that inspects build logs, generates scripts, or autonomously "fixes" a failing build becomes an ungoverned privileged operator. A prompt-injection payload hidden in source, a README, or a build log can change how it behaves.

**Defensive control.** Keep long-lived secrets out of prompts, logs, and build environments. Run AI-connected jobs in isolated environments with tightly scoped, ephemeral credentials, and require policy checks before any pipeline can alter infrastructure or release software. In Kubernetes-native CI, an admission policy is a hard gate that an agent cannot talk its way past: a Kyverno or OPA/Gatekeeper rule that rejects unsigned images, for instance, holds regardless of what a compromised pipeline job tries to push.

### Artifact registry and supply chain

AI-generated code can pull in insecure packages, weak configurations, or dependencies nobody reviewed. An assistant may recommend a base image or copy a snippet without checking provenance, licensing, vulnerabilities, or maintenance status. If you cannot establish what went into an image, who approved it, and whether it passed a gate, fast delivery is just unmanaged risk.

**Defensive control.** Require image scanning, SBOM generation, signed artifacts, and promotion gates, and hold AI-generated code to the same review and release bar as human-written code. A practical open source chain looks like:

- **Trivy** or **Grype** to scan images and IaC for known vulnerabilities.
- **Syft** to generate an SBOM per artifact.
- **Cosign** (Sigstore) to sign images and attach attestations.
- **in-toto** to capture signed attestations for each pipeline step, the foundation of SLSA-style provenance.
- **The Update Framework (TUF)** and **Notary/Notation** to secure and verify artifact distribution.

### Continuous delivery

An AI release agent may be able to edit Helm charts, update GitOps manifests, change deployment targets, approve releases, or trigger rollbacks. Connected to production without boundaries, it can bypass exactly the change-management controls a mature process depends on.

**Defensive control.** Draw a hard line between an agent that *recommends* a release action and one that *executes* it. High-impact actions should sit behind explicit approval gates with strong audit trails: production deployment, privilege escalation, data export, deletion, network-policy changes. In a GitOps model, the pull request *is* the approval gate: the agent proposes a manifest change, a human approves the merge, and the controller reconciles. Enforce that no path to production exists that skips the gate.

### Kubernetes runtime

This stage is often the most consequential. A remediation agent gets `cluster-admin` "temporarily" to investigate an alert, and a convenience tool becomes a high-value target. A compromised or manipulated agent with broad permissions can enumerate secrets, deploy a malicious workload, alter network policies, or exfiltrate data.

**Defensive control.** Namespace boundaries, workload identity, minimal RBAC, admission control, runtime detection, and network segmentation. An agent responsible for restarting a Deployment in one namespace should not be able to read cluster-wide secrets. Concretely:

- **SPIFFE/SPIRE** to give each workload (including agents) a cryptographic identity instead of a shared, long-lived token.
- Least-privilege **RBAC** scoped to a namespace and a verb set, never `cluster-admin`.
- **Falco** or **Tetragon** (an eBPF component of Cilium) for runtime detection of the behavior that follows a compromise: a shell in a container, unexpected secret reads, outbound connections.
- **Cilium** network policies to segment east-west traffic and contain lateral movement.

For example, an agent whose only job is to restart a Deployment needs a namespaced `Role`, not a cluster-wide one: bound to a single namespace, limited to the `apps` API group and the `deployments` resource, and granting only `get`, `list`, and `patch` (a rollout restart is a patch, so `create` and `delete` are never needed). That is a world away from `cluster-admin`, and it means a compromised or manipulated agent cannot read secrets, touch other namespaces, or delete workloads.

## Controls that scale

None of this is about slowing AI adoption. The point is to make AI use visible, accountable, and proportionate to its risk.

### Build an AI inventory

Keep a living inventory of approved and discovered AI tools, models, extensions, code assistants, agents, API integrations, and MCP servers. For each entry, record the business and technical owner, purpose and users, allowed data classification, connected systems and permissions, model or provider, production status, last security review, and the revocation process. You cannot govern what you cannot see, and this is the one control that everything else depends on.

### Treat AI agents as identities

Every agent should have a unique identity. It should never borrow a developer's personal credentials or a shared admin account. Least privilege by default means:

- Separate credentials per environment
- Short-lived tokens, not persistent secrets (SPIFFE/SPIRE, cert-manager, Gitsign)
- Read-only access where possible
- Namespace-scoped Kubernetes permissions
- Explicit allowlists for APIs, tools, repositories, and MCP servers
- Immediate revocation

**Keycloak** can anchor human and service identities for the tooling around the pipeline, while **SPIFFE/SPIRE** and **cert-manager** cover in-cluster workload identity.

### Separate advice from execution

Not every AI function needs the same control level. Match the control to the blast radius:

| AI capability | Recommended control level |
| :-- | :-- |
| Code explanation or documentation drafting | Approved tool, data-classification rules, developer training |
| Code suggestion | Human review, standard testing, secret scanning, dependency checks |
| Pull-request creation | Restricted repo permissions, mandatory peer review |
| CI/CD pipeline modification | Isolated execution, policy-as-code checks, approval gate |
| Cluster remediation | Strict namespace RBAC, limited scope, full audit trail, human approval for high-impact actions |
| Autonomous production changes | Exceptional approval only, time-bound access, kill switch, continuous monitoring |

### Apply defense in depth

Prompt injection is a real risk, but prompt filtering is only one layer. A mature architecture combines governance, identity and access management, source-control protections, secrets management, secure CI/CD, dependency and container scanning, Kubernetes admission and runtime policy, network controls, and centralized logging. The principle is simple: if one control fails, the agent must still be unable to cause material harm.

## Tooling by area (CNCF and open source)

No single project solves Shadow AI. Pick tools by the risk area you need to improve, then integrate them into a coherent architecture. Maturity levels below reflect the CNCF Landscape at the time of writing.

| Area | CNCF projects | Other open source | What it addresses |
| :-- | :-- | :-- | :-- |
| Kubernetes policy & admission | Kyverno (graduated), Open Policy Agent / Gatekeeper (graduated), Kubescape (incubating) | — | Enforce deployment policy, block unsigned or non-compliant workloads, reduce permissions |
| Runtime detection | Falco (graduated), Tetragon / Cilium (graduated) | — | Detect suspicious runtime behavior: shells, secret reads, unexpected egress |
| Network segmentation | Cilium (graduated) | — | Segment east-west traffic, contain lateral movement |
| Workload & agent identity | SPIFFE/SPIRE (graduated), cert-manager (graduated), Keycloak (incubating) | — | Give agents a scoped, short-lived identity; enable least privilege and revocation |
| Supply chain: signing, SBOM, provenance | in-toto (incubating), The Update Framework (graduated), Notary/Notation (incubating) | Sigstore/Cosign & Gitsign, Trivy, Syft, Grype | Scan code and images, generate SBOMs, sign artifacts and commits, prove provenance |
| Secrets management | OpenBao (sandbox), SOPS (sandbox), External Secrets Operator (sandbox) | Sealed Secrets | Central secrets engine, dynamic/short-lived credentials, keep tokens out of repos, prompts, logs, and images |

It is worth being blunt about the gaps. The pipeline and runtime layers above are well served by mature CNCF projects. The thinnest area today is AI governance itself: prompt inspection, agent discovery, tool-call policy. No graduated CNCF project yet owns that space. In practice, teams pair OWASP guidance with a custom policy proxy in front of model and tool endpoints, then lean on the identity and admission controls above to constrain what an agent can *do*, even when they cannot fully inspect what it is *thinking*.

Open source controls are a credible starting point for technically mature teams, but they do not remove the need for ownership. Someone has to maintain the policies, triage the findings, manage the integrations, and prove the controls actually work.

## Conclusion

Shadow AI is the next evolution of Shadow IT, with a critical difference: modern AI agents can interpret information, call tools, and act across engineering systems. That makes them powerful accelerators and potential high-speed paths to source-code leakage, credential compromise, supply-chain abuse, and production disruption.

The decision is not whether your developers will use AI. They already are. The decision is whether you manage AI as an untracked pile of productivity tools or as a governed set of identities, data flows, and production-capable workloads.

The path forward is concrete, and most of it is already covered by cloud-native open source: discover AI use, assign ownership, scope permissions with real workload identity, protect the supply chain with signing and provenance, enforce Kubernetes boundaries with admission and runtime policy, segment the network, monitor behavior, and keep a human in the loop for consequential actions. Let agents help teams move faster, but never let them operate beyond your ability to see, control, and stop them.

## References

- [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)
- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)
- [CNCF Landscape](https://landscape.cncf.io/)
- [in-toto](https://in-toto.io/)
- [The Update Framework (TUF)](https://theupdateframework.io/)
- [SLSA: Supply-chain Levels for Software Artifacts](https://slsa.dev/)
