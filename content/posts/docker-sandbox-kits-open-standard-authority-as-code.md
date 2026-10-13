---
title: "Docker Sandbox Kits: An Open Standard for Authority as Code"
date: 2026-10-13T00:00:00Z
tags: [
  "docker", "sandbox-kit", "authority-as-code", "open-standard",
  "workload-security", "security", "container-security", "devsecops",
  "cncf", "oci", "ai-agents", "supply-chain"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Docker Sandbox Kits are an open standard for authority as code: one OCI image carries a workload, its network policy and credentials. Planned for CNCF."
keywords: [
  "Docker Sandbox Kits", "authority as code", "open standard",
  "Docker Sandbox Kit specification", "Sandbox Kit Spec v3",
  "OCI image authority as code", "workload security",
  "Sandbox Kit CNCF governance", "AI agent sandboxing",
  "network policy credentials", "Docker sbx"
]
canonicalURL: "https://www.msbiro.net/posts/docker-sandbox-kits-open-standard-authority-as-code/"
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
    alt: "Docker Sandbox Kits: an open standard for authority as code, packaged in an OCI image"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In April I tested Docker Sandboxes on my MacBook and wrote up the results in [Docker Sandboxes: Running AI Agents in YOLO Mode, Safely](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/). The boundary held, but the setup around it did not travel with the agent. Network allows, secrets, and mounts stayed in my shell history.

Docker has now named that missing piece. At WeAreDevelopers in September 2026 it announced the [Docker Sandbox Kit Specification v3](https://www.docker.com/blog/docker-sandbox-kit-spec/), an open standard for declaring what a workload may do, open source under Apache 2.0 at [docker/sandbox-kit-spec](https://github.com/docker/sandbox-kit-spec), with the intent to bring it to the CNCF under neutral governance ([partnership announcement](https://www.docker.com/blog/docker-sandbox-kit-spec-cncf/)).

The split is clean. MCP standardized how an agent talks to a tool. Kits standardize how the whole arrangement ships, agent plus tools plus requested access, in one image anyone can pull.

I read the spec, both announcement posts, and the repo. I have not authored a Kit yet, so this is my reading as an `sbx` user, focused on security reviews.

## The half a Dockerfile never covered

Christian Dupuis states the problem plainly in [From Dockerfile to Kit](https://www.docker.com/blog/docker-sandbox-kit-spec/). A Dockerfile describes the inside of the image completely: how software builds, what gets packaged, how it starts. OCI then standardized the image it produces and how registries distribute it.

The outside stayed unwritten. Networks, credentials, volumes, tools, context. That half lived in `docker run` flags, a Compose file, a CI config, an onboarding doc, plus the memory of whoever set it up. Outside the artifact, unversioned, unreviewable.

Agents made that gap urgent. In my [sbx test](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/) the working Copilot setup needed extra `sbx policy allow network` rules for Copilot and telemetry hosts, a global `sbx secret set -g github`, and a temporary token export. It worked. A teammate pulling the same template would still have to reconstruct all of it.

A Kit moves that second half into the artifact. Content plus the authority it asks for, in one image, under one digest. That is authority as code.

## Kits are ordinary OCI images

A Kit introduces no new media type, no artifact type, no sidecar file. The manifest carries declarations in one annotation, `vnd.docker.sandbox.kit.descriptor`. The layers carry content.

That choice removes adoption friction. A Kit builds with `docker buildx build`, pulls with `docker pull`, and passes through the scanners and signers you already run. It also works in a `FROM`. Pin the digest and you pin policy, content, and metadata together.

There are two composable kinds:

* `workload` supplies the root filesystem. Exactly one per composition.
* `mixin` is an overlay on that filesystem. A CLI with its network rule, a credential binding, context for an agent. Zero or more per composition.

A third form, `kind: set`, names other Kits in a `kits:` list and merges them into one publishable Kit. Coherence is checked at build time. An incoherent set fails at your build, not at someone else's launch.

If you have used `sbx`, you have already used Kits. Version 3 is the specification described here. The `docker` org on Docker Hub now publishes v3 Kits, while the older `sbx` org still carries the v2 line, so do not mix the two.

## Open by design, headed for CNCF

Docker frames this as OCI again. Ten years ago Docker donated its image format and runc, OCI formed around them, and an image built anywhere runs anywhere. The spec post states the reasoning directly: "Docker maintains the specification today, and it should not stay under a single vendor: a format for deciding what an agent may do is worth less if it belongs to whoever sells you the runtime. Docker Sandboxes will be a first-class implementation, not the only one."

Where things stand today: the spec is not a CNCF project yet. The repo's `GOVERNANCE.md` still says Docker maintains it, and neither post names a CNCF stage. What exists is the commitment, and CNCF's own position, stated by its CTO in the partnership post: "OCI is the foundation the cloud native ecosystem is built on, so a standard for agents that builds on OCI reaches the whole ecosystem at once. The CNCF welcomes this, and we're excited to work with Docker and the community on making it broadly adopted."

Docker says it worked with AWS, Box, Datadog, Dynatrace, JFrog, NanoClaw, OpenClaw, Palo Alto Networks, Snyk, and others to build Kits for their tools. The relationship with CNCF is not new either: Docker has been a [CNCF Project Services Provider since 2025](https://www.docker.com/blog/docker-cncf-partnership/), although that agreement predates Kits.

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

Both sides are testable. `kit-tck validate <ref>` checks the artifact, and a runtime adapter suite checks runtime behaviour against the capability pages. The checks ship with the spec, so conformance stays verifiable instead of assumed.

The model matches what I observed in April. Inside `sbx`, API keys appeared as `proxy-managed` placeholders in `env`, while the host proxy injected real values at request time toward `api.github.com`. Kits write that arrangement down declaratively, so you review it before you run it. Background on the isolation boundary is in my [microVMs versus containers comparison](https://www.msbiro.net/posts/microvms-vs-containers-ai-security/).

## Composition that stays reproducible

Container images never solved multiple inheritance. A Dockerfile stage has one `FROM`. Mixins take another path. They order by the dependency graph declared through `provides` and `requires`, never by flag order. The same set always composes to the same image.

The resolver stays strict on purpose:

* Every `requires` must be satisfied inside the set or resolution fails. Nothing is fetched silently to cover a gap.
* Exactly one workload is allowed.
* Two Kits providing the same name fail instead of one shadowing the other.
* Overlapping declarations reconcile by rule. Network rules union, hooks run in dependency order, guidance merges into one document, licenses union. Incompatible requests are an error.

That strictness is a security control. Silent shadowing and implicit fetching are how overly broad authority normally creeps in.

It also gives a team a clean baseline. One published `kind: set` that merges the workload with the approved mixins means every engineer starts from the same reviewed authority, not from their own collection of flags.

## Upgrades as authority diffs

A version bump in a Kit can mean two different things. It can be a software update. It can also be a change in authority, for example one more host or one more credential. The second case needs a human decision.

In a Kit world that change appears as added lines in a pull request, where it can be reviewed, refused, and tied to the digest.

The spec adds a second gate that does not rely on someone reading the diff. Every descriptor reduces to a normalized set of everything the host would have to grant, and a runtime that gates updates records that set. A version inside the granted set may apply without asking, while any widening stops and asks.

Removing a deny rule counts as widening. If a later `gh` Kit dropped `DELETE /repos/**`, the runtime would hold the upgrade.

This only works because declarations live in the artifact. A separate file describing an image becomes a second source of truth, and second sources drift.

In practice that means two habits for a team. Pin Kits by digest in your docs, the same way you pin images. Treat a Kit upgrade that widens the grant set as an authority change that needs sign-off, not a routine dependency bump.

## Beyond agents to regular workloads

Agents forced the issue because the authority they request is so large. Regular workloads have the same gap in smaller print. The endpoints a service calls, the credentials it needs, the volume that must survive a restart. Today that knowledge lives in a Helm chart, a runbook, or a colleague.

The spec post is explicit about the scope: "This specification is where any software writes down what it needs from the world around it; agents were the case urgent enough to have it built."

The repo already backs part of that. The capability pages define two groups of types:

* Generic: `network-policy@1` and `@2`, `credential@1`, `volume@1`, `port@1`, `usb-device@1`, `host-mount@1`, `resources@1`, `lifecycle@1`.
* Agent-specific: `acp@1`, `agent-skill@1`, `agent-context@1`, `agent-sessions@1`.

Each is typed, versioned, and reviewed as part of the image.

One caveat: both announcement posts frame Kits around agents, so the first wave of Kits targets agent setups. Kits for ordinary services are the spec's stated intent, not something shipped yet.

The win is traceability. What may this workload do has one answer. A teammate can pull it, a reviewer can diff it, a conforming runtime can enforce it.

## Hands-on entry point

The spec, capability pages, and a worked tour live in [docker/sandbox-kit-spec](https://github.com/docker/sandbox-kit-spec). To try it with `sbx`:

```bash
# macOS
brew trust docker/tap
brew install docker/tap/sbx
sbx login

# run a published workload plus a mixin, no build needed
sbx run docker/sbx-kit-shell:1.0.0 --kit docker/sbx-kit-claude-mixin:2.1.281 .

# local loop from the spec repo, no registry needed
cd examples
sbx run ./hello --kit ./gh .
```

Before trusting a published Kit, read what it asks for with `kit-tck inspect <ref>`. Kits make the network gate auditable, but they do not replace the `--branch` checkpoint from my [sbx article](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/). Keep reviewing what an agent changes before it merges.

If a Kit you need cannot be expressed, or a runtime duty cannot be implemented as stated, open an issue. That feedback is the stated purpose of the experimental phase before the final version targeted for Q4 2026.

## My take

I like this initiative, and not because it is new. It fills a gap I have worked around for years. Every security review I have done hits the same wall at some point: the image is scanned, signed and pinned, but what the workload is allowed to reach lives somewhere else. Kits put that answer next to the content, in a format reviewers can diff and runtimes can enforce.

The choices behind it are what make me take it seriously. Plain OCI instead of a new artifact type, Apache 2.0, conformance suites in the repo, and a public commitment to neutral governance under CNCF. A permission format controlled by one runtime vendor would be one more lock-in. An open one can become shared infrastructure, the way the image format did.

What I am waiting to see is adoption outside Docker's own tooling. A second conforming runtime. Agent clients and CI systems that read the descriptor instead of their own config. Kits for ordinary services, not only agent setups. And Kit declarations showing up in the policy tools we already run on clusters. None of that exists yet, and the spec could still change before the final version. But if it lands, the authority a workload asks for stops being tribal knowledge and becomes part of the artifact, wherever it runs.

As the spec post puts it: "Dockerfiles made software reproducible. Kits make authority reproducible."

---

*Sandbox Kit Specification v3 is experimental as of September 2026. Docker Sandboxes (`sbx`) is the first conforming runtime. Check the [spec repository](https://github.com/docker/sandbox-kit-spec) for current status.*

---

## References

* [From Dockerfile to Kit: the Docker Sandboxes Kit Specification](https://www.docker.com/blog/docker-sandbox-kit-spec/)
* [Docker and CNCF partner on an open spec for agent permissions](https://www.docker.com/blog/docker-sandbox-kit-spec-cncf/)
* [Docker and CNCF: Partnering to Power the Future of Open Source](https://www.docker.com/blog/docker-cncf-partnership/)
* [Docker Sandbox Kit Specification v3 (docker/sandbox-kit-spec)](https://github.com/docker/sandbox-kit-spec)
* [Docker Sandboxes: Running AI Agents in YOLO Mode, Safely](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/)
