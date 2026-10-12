---
title: "Docker Sandbox Kits: From Dockerfile to Authority as Code"
date: 2026-10-13T00:00:00Z
tags: [
  "docker", "ai-agents", "security", "container-security", "devsecops",
  "cncf", "oci", "sandbox", "sandbox-kit", "supply-chain"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Docker Sandbox Kit Spec v3 packages agent, tools, network policy and credentials as one OCI image. Why authority as code matters for AI agents."
keywords: [
  "Docker Sandbox Kit", "Sandbox Kit Spec v3", "Docker sbx",
  "OCI image authority as code", "AI agent sandboxing",
  "CNCF Sandbox Kit", "network policy credentials"
]
canonicalURL: "https://www.msbiro.net/posts/docker-sandbox-kit-authority-as-code/"
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
    alt: "Docker Sandbox Kit Spec v3 packages authority as code in an OCI image"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In April I tested Docker Sandboxes on my MacBook and wrote up the results in [Docker Sandboxes: Running AI Agents in YOLO Mode, Safely](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/). The boundary held, but the setup around it did not travel with the agent. Network allows, secrets, and mounts stayed in my shell history.

Docker has now named that missing piece. At WeAreDevelopers in September 2026 it announced the [Docker Sandbox Kit Specification v3](https://www.docker.com/blog/docker-sandbox-kit-spec/), open source under Apache 2.0 at [docker/sandbox-kit-spec](https://github.com/docker/sandbox-kit-spec), with the intent to bring it to the CNCF under neutral governance ([partnership announcement](https://www.docker.com/blog/docker-sandbox-kit-spec-cncf/)).

The parallel is deliberate. Ten years ago Docker donated its image format and runc, OCI formed around them, and an image built anywhere runs anywhere. A standard for what an agent may do loses value if it belongs to whoever sells the runtime.

The CNCF link continues the [DSOS and service provider partnership from 2025](https://www.docker.com/blog/docker-cncf-partnership/). Early builders include AWS, Box, Datadog, Dynatrace, JFrog, NanoClaw, OpenClaw, Palo Alto Networks, and Snyk.

The split is clean. MCP standardized how an agent talks to a tool. Kits standardize how the whole arrangement ships, agent plus tools plus requested access, in one image anyone can pull.

I read the spec, both announcement posts, and the repo. I have not authored a Kit yet, so this is my reading as an `sbx` user, focused on security reviews.

## The half a Dockerfile never covered

Christian Dupuis states the problem plainly in [From Dockerfile to Kit](https://www.docker.com/blog/docker-sandbox-kit-spec/). A Dockerfile describes the inside of the image completely: how software builds, what gets packaged, how it starts. OCI then standardized the image it produces and how registries distribute it.

The outside stayed unwritten. Networks, credentials, volumes, tools, context. That half lived in `docker run` flags, a Compose file, a CI config, an onboarding doc, plus the memory of whoever set it up. Outside the artifact, unversioned, unreviewable.

Agents made that gap urgent. In my [sbx test](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/) the working Copilot setup needed extra `sbx policy allow network` rules for Copilot and telemetry hosts, a global `sbx secret set -g github`, and a temporary token export. It worked. A teammate pulling the same template would still have to reconstruct all of it.

A Kit moves that second half into the artifact. Content plus the authority it asks for, in one image, under one digest.

## Kits are ordinary OCI images

A Kit introduces no new media type, no artifact type, no sidecar file. The manifest carries declarations in one annotation, `vnd.docker.sandbox.kit.descriptor`. The layers carry content.

That choice removes adoption friction. A Kit builds with `docker buildx build`, pulls with `docker pull`, and passes through the scanners and signers you already run. It also works in a `FROM`. Pin the digest and you pin policy, content, and metadata together.

There are two composable kinds:

* `workload` supplies the root filesystem. Exactly one per composition.
* `mixin` is an overlay on that filesystem. A CLI with its network rule, a credential binding, context for an agent. Zero or more per composition.

A third form, `kind: set`, names other Kits in a `kits:` list and merges them into one publishable Kit. Coherence is checked at build time. An incoherent set fails at your build, not at someone else's launch.

If you have used `sbx`, you have already used Kits. Version 3 keeps the idea and changes the packaging to plain OCI. The `docker` org on Docker Hub now publishes v3 Kits, while the older `sbx` org still carries the v2 line, so do not mix the two.

## A permission slip you can read

The GitHub CLI mixin from the spec post convinced me. It is short enough to review in one glance:

```yaml
capabilities:
  - type: com.docker.sandbox/network-policy@2
    config:
      runtime:
        allow:
          - github.com
          - hosts: [api.github.com]
            methods: [GET, HEAD, POST, PATCH, PUT, DELETE]
        deny:
          - hosts: [api.github.com]
            methods: [DELETE]
            paths: [/repos/**]
  - type: com.docker.sandbox/credential@1
    optional: true
    config:
      service: github
      phase: runtime
      apiKey:
        name: GH_TOKEN
        proxyManaged: true
        inject:
          - {domain: api.github.com, header: Authorization, format: "Bearer %s"}
```

This Kit asks to reach GitHub and nowhere else. It asks for most of the API but not deletes under `/repos/**`, because deny wins. The token that can open a pull request cannot delete the repository.

Two words do the heavy lifting here: *asks* and *conforming*. A Kit grants itself nothing. Each entry is a request and the host decides. A conforming runtime implements the behaviour the capability page describes and blocks hosts not on the list. Without one, the annotation is inert. Docker Sandboxes is the first conforming runtime.

The model matches what I observed in April. Inside `sbx`, API keys appeared as `proxy-managed` placeholders in `env`, while the host proxy injected real values at request time toward `api.github.com`. Kits write that arrangement down declaratively, so you review it before you run it. Background on the isolation boundary is in my [microVMs versus containers comparison](https://www.msbiro.net/posts/microvms-vs-containers-ai-security/).

## Composition that stays reproducible

Container images never solved multiple inheritance. A Dockerfile stage has one `FROM`. Mixins take another path. They order by the dependency graph declared through `provides` and `requires`, never by flag order. The same set always composes to the same image.

The resolver stays strict on purpose:

* Every `requires` must be satisfied inside the set or resolution fails. Nothing is fetched silently to cover a gap.
* Exactly one workload is allowed.
* Two Kits providing the same name fail instead of one shadowing the other.
* Overlapping declarations reconcile by rule. Network rules union, hooks run in dependency order, guidance merges into one document, licenses union. Incompatible requests are an error.

That strictness is a security control. Silent shadowing and implicit fetching are how overly broad authority normally creeps in.

## Upgrades as authority diffs

A version bump in a Kit can mean two different things. It can be a software update. It can also be a change in authority, for example one more host or one more credential. The second case needs a human decision.

In a Kit world that change appears as added lines in a pull request. Reviewable, refusable, tied to the digest.

The spec adds a second gate that does not rely on someone reading the diff. Every descriptor reduces to a normalized set of everything the host would have to grant, and a runtime that gates updates records that set. A version inside the granted set may apply without asking, while any widening stops and asks.

Removing a deny rule counts as widening. If a later `gh` Kit dropped `DELETE /repos/**`, the runtime would hold the upgrade.

This only works because declarations live in the artifact. A separate file describing an image becomes a second source of truth, and second sources drift.

Both sides are testable. `kit-tck validate <ref>` checks the artifact, and a runtime adapter suite checks runtime behaviour against the capability pages. The checks ship with the spec, so conformance stays verifiable instead of assumed.

## Beyond agents to regular workloads

Dupuis closes with the line I keep returning to: Dockerfiles made software reproducible, Kits make authority reproducible.

Agents forced the issue because the authority they request is so large. Regular workloads have the same gap in smaller print. The endpoints a service calls, the credentials it needs, the volume that must survive a restart. Today that knowledge lives in a Helm chart, a runbook, or a colleague.

If the spec holds, the declarative phase gets stronger for everything. Network policy, credential bindings, volumes, ports, devices, lifecycle hooks, all typed and versioned (`network-policy@1` and `@2` both exist today), all reviewed as part of the image.

## Running this in a security team

My plan as a team lead, not a field report yet:

1. Start from published Kits (`sbx run docker/sbx-kit-shell:1.0.0 --kit docker/sbx-kit-claude-mixin:2.1.281 .`) and inspect with `kit-tck inspect` before trusting them.
2. Pin Kits by digest in team docs, same as images. Treat Kit upgrades as authority changes, with signoff when the grant set widens.
3. Publish one team set (`kind: set`) that merges workload plus approved mixins, so every engineer starts from the same place.
4. Keep the `--branch` workflow from my [sbx article](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/). The microVM is the boundary, the proxy is the gate, the branch is the checkpoint. Kits make the gate auditable. They do not replace the checkpoint.

The win is traceability. What may this agent do has one answer. A teammate can pull it, a reviewer can diff it, a conforming runtime can enforce it.

## Hands on entry point

The spec, capability pages, and a worked tour live in [docker/sandbox-kit-spec](https://github.com/docker/sandbox-kit-spec). `sbx` is the first conforming runtime:

```bash
# macOS
brew install docker/tap/sbx
sbx login

# run a published workload plus a mixin, no build needed
sbx run docker/sbx-kit-shell:1.0.0 --kit docker/sbx-kit-claude-mixin:2.1.281 .

# local loop from the spec repo, no registry needed
cd examples
sbx run ./hello --kit ./gh .
```

If a Kit you need cannot be expressed, or a runtime duty cannot be implemented as stated, open an issue. That feedback is the stated purpose of the experimental phase before the final version targeted for Q4 2026.

---

*Sandbox Kit Specification v3 is experimental as of September 2026. Docker Sandboxes (`sbx`) is the first conforming runtime. Check the [spec repository](https://github.com/docker/sandbox-kit-spec) for current status.*

---

## References

* [From Dockerfile to Kit: the Docker Sandboxes Kit Specification](https://www.docker.com/blog/docker-sandbox-kit-spec/)
* [Docker and CNCF partner on an open spec for agent permissions](https://www.docker.com/blog/docker-sandbox-kit-spec-cncf/)
* [Docker and CNCF: Partnering to Power the Future of Open Source](https://www.docker.com/blog/docker-cncf-partnership/)
* [Docker Sandbox Kit Specification v3 (docker/sandbox-kit-spec)](https://github.com/docker/sandbox-kit-spec)
* [Docker Sandboxes: Running AI Agents in YOLO Mode, Safely](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/)
