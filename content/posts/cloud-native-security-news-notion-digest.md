---
title: "Collecting Cloud Native Security News Without Reading Everything"
date: 2026-11-24T08:00:00Z
tags: ["cybersecurity", "devsecops", "cloud-native", "ai-security", "ai-agents"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "I was reading a dozen release pages by hand every week to catch the security-relevant changes. I replaced that with a local-first pipeline: deterministic collectors, a 14B model as judge, and a Notion digest under the page I already open every morning."
canonicalURL: "https://www.msbiro.net/posts/cloud-native-security-news-notion-digest/"
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
    alt: "A local-first pipeline that turns cloud native release feeds into a Notion digest"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Part of my job is knowing what changed in the cloud native and security tooling I work with. Not in a general way. I mean the specifics: which release fixed an auth bypass, which admission controller changed a default, which runtime patched an escape, which CVE touches a component I actually run.

For a while I did this the obvious way. I kept a list of bookmarks and opened them during the week. Kubernetes releases, the official CVE feed, runc, containerd, Falco, Trivy, Vault, the projects around secrets management I follow closely. The list grew as my work moved around. By the end it was around sixty endpoints across eight areas, and I was checking maybe a third of them with any regularity.

That is the wrong way to do it, and I knew it. The problem is not that reading release notes is hard. It is that the important items are a small fraction of the total, they are scattered across sources with no shared format, and you only find out you missed one when a customer asks or when a scanner lights up an environment. My attention was the scheduler, and it was a bad one.

## What I actually wanted

I wrote the requirements down before building anything, because this kind of tool is easy to make useless. The digest had to arrive where I already look, which is a Notion page next to my other work notes. It had to cover sources I choose, not a curated feed someone else decides is relevant. It had to tell me why an item matters, not just that a version number moved; a release title like `v1.9.2` says nothing until you know it patches a container escape.

And it had to be honest about itself. If the pipeline breaks, I want a failure, not a cheerful empty page.

I also did not want a subscription. The obvious shortcut here is to pipe a few RSS feeds into a hosted LLM and call a Notion API. It works, but it sends release text and advisory details to a third party, it costs tokens every run, and it makes the whole thing depend on a service I do not control. Since I already run [Docker Model Runner for local inference](/posts/local-ai-meeting-minutes-docker-agent/), running the model on this Mac was the natural choice.

The result is [cloud-native-security-news-2-notion](https://github.com/matteobisi/cloud-native-security-news-2-notion). It collects, judges, and publishes a digest into Notion, with no cloud model provider involved.

## How it works

The pipeline has three stages, and the separation between them is the most important design decision in the project.

**Collect.** A set of deterministic TypeScript adapters read the endpoints in `config/sources/*.yaml` and normalize whatever they find into a common item shape. Each category is one file, enables itself with an `enabled` flag, and carries a default priority. The active categories today are Kubernetes and container runtimes, security tooling, CI/CD and GitOps, compliance scanning, and container build. Infrastructure as code, databases and the Linux kernel, and observability are in the repository but switched off. I would rather add them when I want them than carry their noise now.

There are four adapter types: `github_release`, `rss`, `json_feed`, and `website`. The Kubernetes official CVE feed is read through the generic JSON Feed adapter, since it is already a JSON Feed document and does not need its own code path.

**Enrich.** A keyword screen runs first. Anything matching the security signals in `config/pipeline.yaml` (CVE, advisory, vulnerability, RBAC, TLS, exploit, secret, supply chain, and a few more) becomes a candidate. Only the candidates reach the model. The model gets batches of eight items and returns, for each one, whether it qualifies as security-relevant news, a severity from one to five, a two sentence summary, and a single sentence on why it matters.

The prompt is deliberately narrow. It says to summarize only what the text says and not to invent CVE numbers or links, which is the failure mode I care about most in this kind of tool.

**Render.** The digest is published as a child page under a parent page I choose, using the Notion REST API. Items come out as collapsible blocks with a severity emoji, a summary, the "why it matters" line, and a bookmark. The top of the page carries a short overview, and anything critical gets a callout.

## The decisions that mattered

**Collection is deterministic, the model only judges.** This is [ADR-0001](https://github.com/matteobisi/cloud-native-security-news-2-notion/blob/main/docs/adr/0001-deterministic-collector-ai-enrichment.md) and it addressed the thing that worried me about a fully agentic design. If the model drives fetching, then HTTP failures, rate limits, and feed quirks become model failures, and debugging that means reading a transcript. Keeping the I/O in code means a broken source fails as a broken source, with an error line naming it. It also keeps adapters testable without a model or a network. That is why enrichment takes an injected `fetchImpl`, and why the state store is injected too.

**The model degrades to rules instead of dying.** If a batch of model calls fails, enrichment falls back to the keyword match that already happened: the item qualifies, severity comes from the source priority, and the summary is the title. The digest is then less useful, not missing. On a pipeline that runs unattended, that is the right trade.

**Notion over REST, not MCP.** I looked at both MCP options and rejected them for reasons that matter for an unattended job. The hosted Notion MCP server needs OAuth or a Business/Enterprise MCP token to run without a human, and the local `@notionhq/notion-mcp-server` is marked unmaintained. The REST API is deterministic, works on any plan, and does not add a protocol hop inside the container. The cost is that requests have to respect the 100 children per call limit and back off on 429 and 529, which is a few lines.

**One endpoint per source.** This came from a real bug. The registry used to allow a `github_repo` type, which was silently read through the releases endpoint, so pointing it at `kubernetes/enhancements` returned nothing at all. A `mailing_list` type was dropped at load time, and with it the Kubernetes security-announce channel. Now every entry is exactly one URL read by exactly one adapter, duplicate endpoints fail the run instead of burning quota, and anything that has two useful feeds becomes two sources.

**Deduplication respects the source.** Canonical URLs dedupe globally, but normalized titles only dedupe within a single source. That looks odd until you have three projects in the same digest shipping `v1.2.0`, which is exactly how you lose two releases.

**Cadence, not a daily dump.** Publishing requires a minimum of three qualifying items, or thirty days since the last digest. Skipped weeks roll forward, and the window never reaches back further than thirty five days. A tool that posts something every day trains you to stop reading it.

## What runs it, and how it fails

`./newsletter run` goes through Docker Agent, which serves two purposes. It is the front door for the project, and it exercises the same model runtime that enrichment uses. The agent's instruction is a single shell call with a long timeout, and its job is to run the deterministic pipeline inside a Compose service.

Getting that to work took a few attempts that are documented in [ADR-0005](https://github.com/matteobisi/cloud-native-security-news-2-notion/blob/main/docs/adr/0005-docker-agent-is-the-run-entrypoint.md), and they are worth knowing if you try something similar. Under `--safety restricted` the agent refused to run `node` at all, because the command is not classified safe and the restricted fallback denies rather than asks. With gemma3 as the agent model, it often printed the command inside a fake `tool_code` block and exited zero without executing anything. Running qwen for the agent and gemma for enrichment at the same time put both models in memory and crashed the Metal backend once. The current setup uses qwen2.5-14b-instruct for both roles, one model resident, and `--safety autonomous`.

An agent that exits zero without doing the work is the dangerous failure, so the wrapper does not trust the exit code. Each run writes `state/last-run.json` with a run id, timestamps, status and exit code, and the wrapper checks that the run id in that file matches the one it generated before it reports success. If the pipeline never ran, the wrapper fails loudly. Failed Notion publishes are saved to `state/pending/` and retried on the next run rather than dropped.

For scheduling, `scripts/run.sh` starts Docker Desktop, waits for the daemon, then calls the same entrypoint. That way the LaunchAgent and my manual runs are the same code path, which is the only way I trust a scheduled job.

## Takeaways

The interesting part of this project was not the model. It was deciding where the model should not be. Collection stayed in code because I need failures I can read, and the model got a narrow job: judge relevance, assign severity, write two sentences, and invent nothing. Everything downstream, from deduplication to cadence to the run marker, exists to keep a mostly-reliable process from lying to me when it fails.

If you build something similar, the two things I would copy are the keyword screen before the model and the run marker after it. The first keeps cost and latency bounded and gives you a fallback; the second turns the worst failure mode, a silent no-op that reports success, into an error.

The code, the source registry, and the ADRs are in [the repository](https://github.com/matteobisi/cloud-native-security-news-2-notion). The source list is the part you will want to change, since it should reflect what you actually run. Mine covers the projects I end up writing about, from the [Kubernetes security cadence](/posts/k8s-security-2026-graduates-2027-preview/) to [hardened image catalogs](/posts/hardened-images-catalog-2026-non-negotiable/) and the [secrets manager landscape](/posts/secrets-manager-catalog-2026-non-negotiable/), which is a decent proxy for what lands on my desk.
