---
title: "SentinelOne Purple MCP: A Hands-On Guide to Singularity AI Integration"
date: 2026-05-11
tags: [
  "cybersecurity", "mcp", "sentinelone", "soc", "ai", "threat-hunting",
  "devsecops", "cloud-native", "kubernetes"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Hands-on review of SentinelOne's purple-mcp: how to connect Singularity alerts, vulnerabilities, and threat hunting to Claude Code for faster SOC triage."
canonicalURL: "https://www.msbiro.net/posts/sentinelone-purple-mcp-singularity/"
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
    alt: "SentinelOne Purple MCP: A Hands-On Guide to Singularity AI Integration"
    caption: "SentinelOne Purple MCP: A Hands-On Guide to Singularity AI Integration"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Every technical support team I have worked with shares the same friction point: an analyst keeps four tabs open simultaneously (the EDR console, a ticketing system, an asset CMDB, and a query window) and spends a sizeable chunk of their shift copy-pasting IDs between them. The intelligence exists. The problem is getting it out fast enough.

The [Model Context Protocol (MCP)](https://modelcontextprotocol.io) is the most credible attempt I have seen yet to reduce that cost. It is a small, open specification for letting LLM-driven assistants invoke external tools in a typed, structured way: a server exposes a catalogue of tools with JSON Schema input contracts, and any MCP-aware client (Claude Desktop, Claude Code, Zed, or your own automation) can call them without writing any glue code. One server definition, every compatible client for free.

The interesting question for security teams is: which vendors are shipping real, production-ready MCP servers? Last week I spent an afternoon evaluating one of the better examples I have encountered: [**purple-mcp**](https://github.com/Sentinel-One/purple-mcp), the open-source MCP server published by SentinelOne for the Singularity platform. I installed it, registered it inside Claude Code, and exercised every tool against a live SentinelOne tenant. This article is the writeup.

> **Repo**: [github.com/Sentinel-One/purple-mcp](https://github.com/Sentinel-One/purple-mcp)  
> **License**: Apache-2.0  
> **Tested version**: PurpleAIMCP v3.2.4, MCP protocol `2025-06-18`  
> **Tested console**: SentinelOne Singularity, build S-26.1.6

---

## What problem does an MCP server actually solve?

The shortest framing: an MCP server makes a backend product addressable from natural language without requiring you to write any per-tool wrappers.

A Claude Code session with purple-mcp loaded can turn a prompt like:

> *"List the 10 most recent critical alerts. For the top one, fetch the full record, the asset details, and any related history."*

into the right sequence of tool calls (`list_alerts`, `get_alert`, `get_inventory_item`, `get_alert_history`) and return a single triage brief. No tab-switching, no copy-pasting IDs.

The concept is not new. Splunk, Elastic, and CrowdStrike all expose HTTP APIs you could orchestrate with a script. What MCP changes is the integration cost: you write zero code in your assistant. The server author defines the surface once, and every MCP-aware client gets it with type checking, structured inputs, and a standard discovery mechanism. For security teams that want to prototype automation without committing to custom tooling, this is a meaningful difference.

For purple-mcp specifically, that surface is read-only access to a SentinelOne tenant. The README is explicit about it:

> *"Purple AI MCP is a read-only service — you cannot make changes to your account or any objects."*

Read-only is the right starting point when you are letting an LLM drive your EDR, and this design choice reflects that.

---

## What purple-mcp actually exposes

The README implies a `purple_utils` namespace that does not exist in v3.2.4. The actual surface, discoverable at runtime via a standard MCP `tools/list` call, is **22 tools** across seven groups:

| Group | Tools | Response shape | Notes |
|---|---|---|---|
| **Helpers** | `get_timestamp_range`, `iso_to_unix_timestamp` | Local, no API call | Timestamp utilities for data-lake queries |
| **Alerts** | `list_alerts`, `search_alerts`, `get_alert`, `get_alert_notes`, `get_alert_history` | GraphQL relay | Filter by severity, status, or timestamp |
| **Misconfigurations** | `list_misconfigurations`, `search_misconfigurations`, `get_misconfiguration`, `get_misconfiguration_notes`, `get_misconfiguration_history` | GraphQL relay | CSPM/KSPM findings; scope with `view_type` (`KUBERNETES`, `CLOUD`, …) |
| **Vulnerabilities** | `list_vulnerabilities`, `search_vulnerabilities`, `get_vulnerability`, `get_vulnerability_notes`, `get_vulnerability_history` | GraphQL relay | CVE records with CVSS scores and exploitability data |
| **Inventory** | `list_inventory_items`, `search_inventory_items`, `get_inventory_item` | REST flat `{data:[…]}` | Different shape from the other groups |
| **PowerQuery** | `powerquery` | SDL response | Requires ISO-8601 timestamps; let Purple AI generate the query |
| **Purple AI** | `purple_ai` | Prose or PowerQuery string | Natural-language interface to the threat-hunting LLM |

Two things worth highlighting before moving on:

- **Inventory response shape.** The inventory tools return a flat REST shape (`{data:[…]}`), while every other group returns the GraphQL relay shape (`{edges:[{node:{...}}], page_info:{...}}`). If you are building parsers or automation on top, plan for both.
- **PowerQuery generation.** Do not write PQ syntax by hand. Ask `purple_ai` the analyst question in plain language, take the generated query from the response, and pass it verbatim to `powerquery`. The dialect is close enough to Splunk SPL to feel familiar, but unforgiving enough that a handwritten `* | limit 5` returned a 500 in my tests.

That is the complete tool surface. No hidden resources, no additional prompts. What `tools/list` returns is what you can do.

---

## Architecture in brief

Under the hood, purple-mcp is a Python 3.10+ package built on [FastMCP](https://github.com/jlowin/fastmcp), a Pythonic MCP server framework. It supports three transports: stdio (the default, used by desktop AI assistants), SSE, and streamable HTTP.

Configuration is via four environment variables: `PURPLEMCP_CONSOLE_TOKEN` and `PURPLEMCP_CONSOLE_BASE_URL` are required; two optional variables override the GraphQL endpoint paths. The server hits two GraphQL endpoints (`/web/api/v2.1/graphql` and `/web/api/v2.1/unifiedalerts/graphql`), the inventory REST surface, and the SDL queries endpoint.

Authentication is via a Site-level (or Account-level) Service User token. Global tokens are explicitly unsupported, which means you need to generate credentials from the right scope before starting.

---

## Installation and wiring into Claude Code

The official install path uses `uvx`, Astral's Rust-based Python toolchain:

```bash
# Bootstrap uv
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"

# Run the server
export PURPLEMCP_CONSOLE_TOKEN="…"
export PURPLEMCP_CONSOLE_BASE_URL="https://your-console.sentinelone.net"
uvx --from git+https://github.com/Sentinel-One/purple-mcp.git \
    purple-mcp --mode=stdio
```

First run downloads roughly 99 transitive dependencies (pandas, numpy, cryptography, FastMCP, Pydantic) and takes about 30 seconds. Subsequent launches are nearly instant thanks to uv's aggressive caching.

For Claude Code, I wrapped the startup in a small script that sources credentials from a sibling `config.txt` file (keeping the API token out of any version-controlled JSON) and strips the trailing slash that Pydantic rejects:

```bash
#!/usr/bin/env bash
set -euo pipefail
HERE="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
set -a; source "${HERE}/config.txt"; set +a
export PURPLEMCP_CONSOLE_BASE_URL="${PURPLEMCP_CONSOLE_BASE_URL%/}"
export PATH="${HOME}/.local/bin:${PATH}"
exec uvx --from git+https://github.com/Sentinel-One/purple-mcp.git \
     purple-mcp --mode=stdio
```

Registration is a single command:

```bash
claude mcp add purple-mcp --scope project -- /path/to/run-purple-mcp.sh
```

Project scope means `.mcp.json` lives in the working directory, so the server only loads when Claude Code opens from that folder. It is a clean way to keep non-production credentials out of unrelated sessions. After a restart, Claude Code prompts once to trust the server, and tools surface as `mcp__purple-mcp__<tool_name>`.

---

## A real session: alert triage with one prompt

Once the server was registered, I typed:

> *"List the 10 most recent alerts. For each: id, severity, asset, and one-line summary."*

Claude called `list_alerts(first=10)`, parsed the GraphQL relay response, deduplicated rule descriptions, and returned a Markdown table. The freshest alerts on my lab tenant were a cluster of Kubernetes runtime detections — container breakout attempt indicators, unexpected shell spawning, API enumeration from within a container — all concentrated on a handful of worker nodes. Two MITRE techniques stood out immediately: T1611 (Escape to Host) and T1059 (Command and Scripting Interpreter).

Total latency from prompt to rendered table: around six seconds. Without the MCP, reaching the same picture involves opening the console, navigating to the alerts view, expanding each row, reading the description, and copying hostnames somewhere else. One prompt replaces those five steps.

The next prompt was more interesting:

> *"For the worker showing container-breakout indicators, run a Purple AI query for any process events in the last hour. Tell me whether anything looks like real exploitation versus simulated red-team noise."*

Claude piped the question through `purple_ai`, which generated a PowerQuery; then ran it via `powerquery` over a one-hour window. The result was a compact table of process events with timestamps, ready for analysis. Five tool calls, zero syntax to remember, no manual time-range formatting.

---

## Where this genuinely helps SOC teams

After exercising every tool, these are the five workflows I think justify the deployment effort.

**Alert triage with context enrichment.** One prompt collapses alert retrieval, asset lookup, history fetch, and analyst note retrieval into a single triage brief. For any SOC triaging dozens of alerts per shift, this is a meaningful time saving.

**Vulnerability-to-asset prioritization.** Cross-reference critical CVEs against the inventory to rank exposure: is the host internet-facing, is it production, how many endpoints are affected? Manual today, scriptable, but tedious. With the MCP it is a single prompt producing a Markdown table you can paste directly into a ticket.

**Kubernetes and cloud misconfiguration hunts mapped to MITRE.** Pull the `KUBERNETES` view of misconfigurations, then ask Purple AI to map each finding to the closest ATT&CK technique. If your SOC tracks MITRE coverage (and most modern playbooks do), this is a compelling workflow.

**Endpoint enrichment during incident response.** "What is host X and what was it doing?" becomes a one-shot question. Particularly useful during a page, when context-switching cost is highest and you cannot afford to spend three minutes navigating the console.

**Ad-hoc threat hunting with `purple_ai` and `powerquery`.** This is the workflow the MCP makes qualitatively better than the console UI. Chat-driven hunting where the model keeps state across questions, and you can iterate on a hypothesis without rewriting queries by hand. A junior analyst who cannot write PowerQuery syntax can still run meaningful hunts by asking natural-language questions.

The first four reduce time on workflows analysts already do every day. The fifth opens up work that was hard or impossible before without a PowerQuery expert in the room.

---

## The rough edges

The project is young. All three published releases landed in November 2025, and the pace suggests it is still finding its shape. That context matters when reading the list below: none of these are design failures, they are the normal friction of early-stage open-source software. Since the repo is Apache-2.0, every item here is also a candidate for a GitHub issue or a small pull request if you have the appetite to contribute.

- **Inconsistent filter shape.** `search_alerts`, `search_misconfigurations`, and `search_vulnerabilities` all expect `filters` as a JSON-encoded array string: `'[{"fieldId":"severity","filterType":"in","value":["CRITICAL"]}]'`. But `search_inventory_items` expects a JSON-encoded dict string: `'{"hostname":"prod-db-01"}'`. Passing an array to inventory search returns a Pydantic validation error with no helpful message. A unified filter contract across all four search tools would fix this cleanly.

- **Service User tokens break user-relative views.** `list_alerts` documents `ASSIGNED_TO_ME`, `UNASSIGNED`, and `MY_TEAM` as valid `view_type` values, but with a Service User token only `ALL` works. A Service User has no human identity, so views scoped to a person fail silently with "Failed to list alerts". Stick to `view_type="ALL"` and filter by severity, status, or timestamp instead.

- **The base URL must not end with a slash.** Pydantic rejects it with a validation error. Easy to hit when copy-pasting a URL from the browser. The wrapper script in the installation section strips it automatically.

- **PowerQuery syntax is unforgiving.** A handwritten `* | limit 5` returned a 500 from the SDL backend. Using `purple_ai` to generate the query first, then passing it verbatim to `powerquery`, worked every time. The README warns about this; it is still easy to skip past on first read.

- **`iso_to_unix_timestamp` returns milliseconds, not seconds, as a string.** Correct for SDL records, but worth knowing upfront if you are used to UNIX timestamps in seconds.

None of these are blockers. Together they cost me about 20 minutes during evaluation. If you want to save the next person those 20 minutes, the issue tracker is open.

---

## Security considerations for multi-user deployments

The read-only guarantee is real and meaningful: no tool in the current surface can mutate your SentinelOne tenant. For a single analyst running Claude Code locally over stdio transport, the token never leaves the workstation and the risk profile is low.

Multi-user deployments need more thought. The MCP server itself does not authenticate its clients; it trusts whoever invokes it. If you expose the server over SSE or HTTP beyond localhost, you need a reverse proxy in front enforcing authentication (OIDC, signed tokens, or mTLS). There is no built-in rate limiting and no audit log at the MCP layer. The token lives in the server process's environment for the lifetime of the process.

For most use cases (an analyst running this locally, or a small team using it inside a controlled environment) these are manageable constraints, not fundamental flaws. For production team deployments, treat the token as a privileged credential and give it the same protection you would give any API key with read access to your EDR.

---

## Takeaways and what I would like to see next

I genuinely enjoyed testing purple-mcp. Even at this early stage, the feature set is well chosen and the integration experience is smooth. The tool surface covers the most important Singularity data (alerts, vulnerabilities, misconfigurations, inventory, and the data lake) without overreaching, and the read-only constraint makes it safe to experiment with. For SOC teams or technical support teams that work daily with Singularity, this is already good enough to slot into an agentic workflow and start getting faster iteration on triage, enrichment, and hunting.

The productivity gains in alert triage and vulnerability prioritization are real and immediate. The threat-hunting workflow (Purple AI generating PowerQueries that you then execute through `powerquery`) is the one that feels most different from working in the console directly.

For a future release, the two things I would like to see most are server-side prompt templates and a unified filter schema. Pre-built prompts for "triage this alert" or "summarize this asset's posture" would lower the bar for analysts who are not used to writing LLM prompts. A single `filters` shape across all four search tools would make automation much easier to build on top.

If you run a SentinelOne tenant, even a non-production one, spending an afternoon with purple-mcp is worth the time. SentinelOne open-sourcing this under Apache-2.0 says something about where the EDR vendor market is heading, and the project has room to grow into something quite useful.

---

## Useful links

- **Repo and docs**: <https://github.com/Sentinel-One/purple-mcp>
- **Model Context Protocol**: <https://modelcontextprotocol.io>
- **MCP specification**: <https://spec.modelcontextprotocol.io>
- **FastMCP** (the framework purple-mcp is built on): <https://github.com/jlowin/fastmcp>
- **Astral uv** (the Python toolchain used for installation): <https://github.com/astral-sh/uv>
- **Claude Code**: <https://claude.ai/code>
- [Securely Working with Third-Party MCP Servers](/posts/securely-using-third-party-mcp-servers/)
- [The Challenge of Securing AI Agents: A DevSecOps Perspective](/posts/securing-ai-agents-devsecops-challenge/)
