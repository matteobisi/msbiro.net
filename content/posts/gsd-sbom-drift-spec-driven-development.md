---
title: "Testing GSD: From a Docs-Only Repo to Working Go Code in One Session"
date: 2026-04-13T07:32:00Z
tags: [
  "devops", "devsecops", "AI", "spec-driven-development",
  "golang", "sbom", "supply-chain-security", "open-source", "github-copilot"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Another SDD experiment: using GSD (Get Shit Done) v1.34.2 with GitHub Copilot and GPT-5.4 to bootstrap sbom-drift from a docs-only repo to working Go code. Installation, project initialization, Phase 1 execution, and honest lessons from the session."
canonicalURL: "https://www.msbiro.net/posts/gsd-sbom-drift-spec-driven-development/"
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
    alt: "GSD: From docs to working Go code in one session"
    caption: "Spec-Driven Development with GSD and GitHub Copilot"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

I have been experimenting with Spec-Driven Development for a while now. If you are not familiar with the approach, I have a few articles tagged [spec-kit](/tags/spec-kit/) that cover the theory and a real hands-on walkthrough where I built a Go TUI for Apple Container management. The short version: instead of vibe-coding with an LLM and hoping for the best, you invest upfront in a structured specification, then let the AI work against that spec. The results are measurably different.

This is another experiment in that space. Different tool, different SDD framework, same honest reporting.

The tool I am building is **sbom-drift**, a CLI to fill a specific gap in supply chain security: existing SBOM tools like Syft and Grype are point-in-time scanners. They answer "what is in this artifact right now?" but not "what changed between this release and the last one, and why should I care?" sbom-drift compares two SBOM files and produces a structured, actionable drift report.

The SDD framework this time is **GSD** (Get Shit Done), a meta-prompting and context engineering system for AI coding agents built by a solo developer called TÂCHES. It is different from Spec-Kit, and the differences matter. More on that below.

The model doing the coding: **GPT-5.4**, via GitHub Copilot.

The starting point: a repo with zero production code. Just architecture diagrams, design decisions, and a memory document. Exactly the kind of "I know what I want to build, now prove that AI can drive it" scenario.

---

## A Brief SDD Recap (For New Readers)

Spec-Driven Development flips the "just start coding" reflex. Instead of describing a feature to an AI and hoping for coherent output, you build a structured specification first: the *what* and *why* before the *how*. That specification becomes the source of truth that guides every implementation decision.

For DevSecOps teams, this matters beyond productivity. AI-generated code without a documented rationale creates audit gaps. Spec-driven workflows produce a paper trail: requirements, architectural decisions, task breakdowns, and implementation records that auditors can follow.

The [spec-kit articles](/tags/spec-kit/) cover the theory and a real implementation session in depth. This one assumes you understand why structured AI workflows are worth the setup cost, and focuses on what GSD specifically brings to the table.

---

## Why GSD: Context Rot and a Lighter Approach

When I finished the Spec-Kit series, I started looking at what else existed in this space. GSD caught my attention for a specific reason: it solves a problem that Spec-Kit does not explicitly address.

**Context rot** is what happens when a long AI session degrades in quality as the context window fills up. The AI starts forgetting earlier decisions, repeating questions it already asked, or drifting away from architectural constraints that were established at the beginning of the session. Anyone who has run a complex implementation session with an LLM has experienced this, even if they did not have a name for it.

GSD's entire design is built around preventing context rot. It does this through three mechanisms:

1. **Meta-prompting**: structured prompts that give the AI exactly what it needs, nothing more
2. **Context engineering**: a planning artifact system that anchors the AI to prior decisions across sessions
3. **Persistent planning state**: a `.planning/` directory of machine-readable files that any GSD session can resume from

The other thing that stands out about GSD is what it deliberately leaves out. The README is explicit: "No enterprise roleplay bullshit. Just an incredibly effective system for building cool stuff consistently." There are no sprint ceremonies, story points, or stakeholder syncs. No concept of a 50-person software company. It is built by a solo developer for people who want to describe what they want and have it built correctly.

That is a very different philosophy from Spec-Kit, which targets structured team governance with a constitution system and explicit audit trails. Both are valid; they serve different needs. GSD is faster to set up, more opinionated about the AI workflow itself, and focused on output quality over documentation ceremony.

---

## GSD Core Concepts

Before the installation walkthrough, it helps to understand how GSD structures its work.

**The workflow** has three phases that repeat per implementation phase:

```
Discuss ──► Plan ──► Execute
  │           │          │
  └── CONTEXT.md  PLAN.md   code + tests
```

- **Discuss** (`/gsd-discuss-phase N`): GSD interviews you about the current phase, locking down decisions that would otherwise become ambiguous later. The result is a `CONTEXT.md` that the planner reads.
- **Plan** (`/gsd-plan-phase N`): GSD spawns research subagents, builds a detailed plan, and runs a plan-checker before presenting the result for approval.
- **Execute** (`/gsd-execute-phase N`): GSD implements the plan, then runs verification including a UAT sequence and a threat review.

**The planning artifacts** live in `.planning/`. They are the external memory that prevents context rot: `PROJECT.md` (what this is and why), `REQUIREMENTS.md` (testable user requirements), `ROADMAP.md` (phase structure), `STATE.md` (current progress), and per-phase files under `phases/`.

**The commands** are delivered as slash commands in your AI coding tool. For GitHub Copilot, they load from `.github/skills/gsd-*/SKILL.md`. The key ones for a new project are:

| Command | Purpose |
|---|---|
| `/gsd-new-project` | Full project initialization: questions → research → requirements → roadmap |
| `/gsd-discuss-phase N` | Gather phase-specific context before planning |
| `/gsd-plan-phase N` | Create detailed phase plan with research and verification |
| `/gsd-execute-phase N` | Implement the plan |
| `/gsd-verify-work N` | Run UAT and threat review for completed phase |
| `/gsd-resume-work` | Resume from a saved pause point |
| `/gsd-help` | Full command reference |

---

## Installation: GitHub Copilot, Local Mode

GSD installs via npm and works with multiple AI runtimes. For GitHub Copilot, the install puts skills into `.github/skills/`, which Copilot loads automatically.

### Prerequisites

- Node.js and npm (v18+ recommended)
- GitHub Copilot in VS Code or another supported editor

### Install Command

Navigate to your project directory and run:

```bash
npx get-shit-done-cc --copilot --local
```

The `--copilot` flag targets the GitHub Copilot runtime. The `--local` flag installs into the current project under `.github/`, keeping the install scoped to this repo. Use `--global` instead to install once and have GSD available across all your projects at `~/.github/`.

This is what the installation looks like:

```
   ██████╗ ███████╗██████╗
  ██╔════╝ ██╔════╝██╔══██╗
  ██║  ███╗███████╗██║  ██║
  ██║   ██║╚════██║██║  ██║
  ╚██████╔╝███████║██████╔╝
   ╚═════╝ ╚══════╝╚═════╝

  Get Shit Done v1.34.2

  Installing for Copilot to ./.github

  ✓ Installed 68 skills to skills/
  ✓ Installed get-shit-done
  ✓ Installed agents
  ✓ Wrote VERSION (1.34.2)
  ✓ Wrote file manifest (gsd-file-manifest.json)
  ✓ Generated copilot-instructions.md
  ✓ Set resolve_model_ids: "omit" in ~/.gsd/defaults.json

  Done! Open a blank directory in Copilot and run /gsd-new-project.
```

68 skills. That is the full GSD command set, covering everything from project initialization to code review, dependency analysis, security verification, and release workflows.

The installer also generates `.github/copilot-instructions.md`, which tells Copilot how to load and execute GSD skills. You do not need to configure anything else.

### Verify the Install

Open Copilot Chat and type:

```
/gsd-help
```

If the install is working you will see the complete GSD command reference printed. If you see nothing or an error, restart VS Code to let Copilot pick up the new `.github/` structure.

---

## Project Initialization: `/gsd-new-project`

sbom-drift had everything documented and nothing coded. The architecture was defined across two files: `docs/architecture.md` with component diagrams and data models, and `docs/memory.md` with design decisions, rationale, and a phased roadmap. I had been careful with this documentation precisely because I knew it would be the input to an AI-driven implementation.

With GSD installed, I opened Copilot Chat and ran:

```
/gsd-new-project
```

### The Context Gathering

GSD's first move is to understand the project through focused questions. It does not ask you to paste a wall of context; it probes with specific, multiple-choice questions designed to extract the decisions that will shape every downstream planning artifact.

**Question 1: Who is the first release mainly for?**

```
1. Security/compliance teams
   Teams comparing release SBOMs for audit and risk review

2. Platform or DevOps teams
   Teams wiring SBOM drift checks into CI/CD gates

3. Developers investigating package changes
   Ad hoc local analysis of release-to-release drift
```

I selected: security/compliance teams. GSD then read the existing docs before asking the next question; it explicitly noted that it had reviewed the repo docs and was targeting the actual scope boundary rather than asking about what the docs had already defined.

**Question 2: For security and compliance teams, what must the first release do exceptionally well?**

```
1. Evidence-ready drift reports
   Clear added, removed, version, license, and dependency changes for audit review

2. Deterministic CI gating
   Reliable machine-readable output and exit codes for pipeline enforcement

3. Transitive risk visibility
   Surface new indirect dependencies and their security or license impact
```

Selected: evidence-ready drift reports.

**Question 3: For v1, what must be strongest?**

```
1. Deterministic offline diffing
2. CI and pipeline gating
3. Human-readable reports
4. Enrichment insight
```

Selected: deterministic offline diffing. This locked the core v1 value proposition.

**Question 4: What stays out of v1?**

```
1. Extra SBOM formats beyond CycloneDX and SPDX
2. Baseline/history workflows like --since
3. Hosted service or dashboard features
```

Selected: extra SBOM formats. Keep the parser quality high by staying focused.

Four questions. GSD now had enough to write `PROJECT.md` without guessing at anything.

### Configuration

After the project questions, GSD asked for workflow preferences in a compact multi-select format. My choices:

| Setting | Choice | Rationale |
|---|---|---|
| Mode | Interactive | I want to approve each phase |
| Granularity | Standard | Balanced detail without over-engineering |
| Execution | Parallel | Let subagents run simultaneously |
| Git tracking | Yes | Keep planning artifacts in version control |
| Research | Research first | Ground requirements in domain knowledge |
| Research agent | Yes | Enable per-phase research subagent |
| Plan check | Yes | Verify plans before execution |
| Verifier | Yes | Confirm deliverables after execution |

These settings get written to `.planning/config.json` and control GSD's behavior for the entire project.

### The Research Phase

With config locked, GSD launched four parallel research subagents before writing requirements. Each subagent focused on a different domain:

- **Stack research**: Go version compatibility, cobra and pflag, encoding/json, CycloneDX Go library, SPDX Go tools
- **Features research**: SBOM format quirks, purl matching strategies, drift detection patterns
- **Architecture research**: CycloneDX and SPDX specs, OSV.dev and deps.dev APIs, OWASP SBOM dependency graph guidance
- **Pitfalls research**: Common parser edge cases, purl inconsistencies, transitive dep graph complexity

Each subagent fetched relevant documentation, read the repo's existing architecture files, and wrote its findings to `.planning/research/`. This took a few minutes but the result was concrete: domain-specific knowledge baked into the requirements rather than generic boilerplate.

### Generated Requirements

GSD presented 21 requirements across five categories for approval before committing:

```
FORM-01: User can diff two CycloneDX JSON SBOM files.
FORM-02: User can diff two CycloneDX XML SBOM files.
FORM-03: User can diff two SPDX JSON SBOM files.
FORM-04: User can diff two SPDX tag-value SBOM files.
FORM-05: User receives a clear error when an input SBOM is malformed,
         unsupported, or missing required comparison data.

DIFF-01: User can see which components were added between the two SBOMs.
DIFF-02: User can see which components were removed between the two SBOMs.
DIFF-03: User can see which matched components changed version.
DIFF-04: User can see license changes for matched components.
DIFF-05: User can see direct and transitive dependency relationship changes.

REPT-01: User can output a deterministic JSON report for the same inputs.
REPT-02: User can output a Markdown report suitable for human review.
REPT-03: User can output a self-contained HTML report for audit handoff.
REPT-04: User can write the report to stdout or a specified output file.

PIPE-01: User can run a CLI command with two explicit SBOM file paths.
PIPE-02: User gets offline-first behavior by default, no external calls.
PIPE-03: Exit code 0 = no drift, 1 = drift found, 2 = execution failure.
PIPE-04: User can fail a pipeline only on selected drift classes.

ENRH-01: User can optionally enrich components with OSV.dev vulnerability results.
ENRH-02: User can optionally enrich components with deps.dev package metadata.
ENRH-03: User still receives the core diff when enrichment providers fail.
```

These were clean and testable. I approved them without changes.

### The Roadmap

GSD generated a 5-phase roadmap immediately after requirements approval:

```
Phase 1: Input Normalization Foundation
Goal: compare supported JSON SBOM inputs through a predictable offline CLI base.
Requirements: FORM-01, FORM-03, FORM-05, PIPE-01, PIPE-02

Phase 2: Core Drift and Deterministic Reporting
Goal: deliver trustworthy component and license drift with JSON and Markdown output.
Requirements: DIFF-01, DIFF-02, DIFF-03, DIFF-04, REPT-01, REPT-02, REPT-04, PIPE-03

Phase 3: Graph Drift and Format Parity
Goal: add dependency graph drift and full v1 format coverage.
Requirements: FORM-02, FORM-04, DIFF-05

Phase 4: Optional Enrichment and Selective Gating
Goal: add opt-in OSV/deps.dev enrichment and targeted pipeline failure.
Requirements: PIPE-04, ENRH-01, ENRH-02, ENRH-03

Phase 5: HTML Audit Handoff
Goal: add self-contained HTML output for audit review.
Requirements: REPT-03
```

This matched exactly what I had sketched in `docs/memory.md`. The roadmap was grounded in the requirements I had just approved, with clear dependencies between phases. I approved it, and GSD committed the full initialization state to git.

---

## Phase 1: From Discussion to Working Code

### Decision Locking: `/gsd-discuss-phase 1`

Phase 1 is "Input Normalization Foundation": get the CLI running, parse CycloneDX JSON and SPDX JSON into a normalized internal model, and match components correctly. Before planning, GSD surfaced the open decisions worth locking:

**Selected for discussion**: CLI shape, identity rules, and parse behavior. I left the mixed-format question open for the planner to decide.

The three decisions locked:

**CLI shape**: `sbom-drift diff <sbom1> <sbom2>` with only essential flags in Phase 1. Single diff subcommand, narrow surface.

**Component identity**: Strict purl-first matching with deterministic fallback to normalized package fields when purl is missing or inconsistent. No fuzzy matching.

**Parse behavior**: Fail closed on invalid or unsupported input. Warnings only for recoverable issues that do not compromise comparison trust.

GSD wrote these decisions to `.planning/phases/01-input-normalization-foundation/01-CONTEXT.md` and a corresponding `01-DISCUSSION-LOG.md`, then committed both.

### Planning: `/gsd-plan-phase 1`

After a `/clear` to reset the Copilot context window, I ran `/gsd-plan-phase 1`. GSD read the project research and the locked context decisions, then produced three plan files:

```
01-01-PLAN.md — Lock the CLI surface, offline execution boundary,
                normalized model, parser contracts, and fail-closed
                error taxonomy.

01-02-PLAN.md — Implement CycloneDX JSON and SPDX JSON normalization
                with deterministic parser behavior.

01-03-PLAN.md — Wire end-to-end comparison, purl-first matching with
                deterministic fallback, and offline integration coverage.
```

The plan-checker ran automatically and validated all three files for structure and completeness. Commit: `8974e32 — docs(01): create phase plan`.

### Execution: `/gsd-execute-phase 1`

I ran `/gsd-execute-phase 1` and let GSD work.

The code that emerged, organized by the plan's three waves:

**Wave 1** locked the project skeleton: `go.mod`, the cobra root command in `cmd/root.go`, the `diff` subcommand in `cmd/diff.go`, and the internal model in `internal/model/sbom.go`. The model defines `SBOM`, `Component`, `Metadata`, and `Dependency` structs along with `identity.go` for the purl-first matching logic.

**Wave 2** implemented the parsers: `internal/parser/cyclonedxjson.go` and `internal/parser/spdxjson.go`, a format detector in `internal/parser/detect.go`, a unified entry point in `internal/parser/parser.go`, and a centralized error taxonomy in `internal/parser/errors.go`. Each parser normalizes its format's component representation into the internal model.

**Wave 3** wired end-to-end comparison: `internal/differ/matcher.go` implements the purl-first matching algorithm with normalized fallback, `internal/app/run.go` connects the parser and differ into a single execution path, and tests were created alongside fixtures in `tests/fixtures/`.

The test suite ran green. Two commits closed the phase:
- `66b6a8f — feat(01): implement offline SBOM normalization and diff foundation`
- `1aba29a — docs(01): record phase 1 execution summaries`

---

## Verification: The UAT Sequence

### `/gsd-verify-work 1`

GSD created a `01-UAT.md` file with five sequential tests and ran me through them interactively. For each test, it described what to run and what to expect, waited for me to report `pass` or paste actual output, then advanced the state file before presenting the next test.

**Test 1: Cold Start Smoke Test**

```bash
go run . diff tests/fixtures/cyclonedx-valid.json tests/fixtures/spdx-valid.json
```

```json
{
  "source_formats": { "left": "cyclonedx-json", "right": "spdx-json" },
  "component_counts": { "left": 1, "right": 1 },
  "matched_by_purl": 1,
  "matched_by_fallback": 0,
  "unmatched_left": [],
  "unmatched_right": [],
  "warnings": []
}
```

Cross-format diffing works from a clean shell. Pass.

**Test 2: Supported JSON Diff Output**

Same-format diff, both inputs as CycloneDX JSON. Deterministic output, no parse warnings. Pass.

**Test 3: Mixed-Format Alignment**

CycloneDX JSON vs SPDX JSON: the `source_formats` field correctly reflects the mixed input and matching counts are stable. Pass.

**Test 4: Malformed Input Failure Contract**

```bash
go run . diff tests/fixtures/cyclonedx-json/malformed.json tests/fixtures/spdx-valid.json
Error: detect left SBOM: detect format: malformed input
exit status 2
```

Fail-closed behavior confirmed: clear error, no partial JSON, no cobra usage text. Pass.

**Test 5: Missing Comparison Data**

```bash
go run . diff tests/fixtures/cyclonedx-valid.json tests/fixtures/spdx-json/missing-comparison-data.json
Error: parse right SBOM: normalize SPDX package "broken-package": missing comparison data
exit status 2
```

The SPDX parser correctly rejects a package missing comparison-critical fields. Pass.

**Result: 5 passed, 0 issues.**

There was one recurring friction during this sequence: GSD's suggested test commands consistently omitted the `tests/fixtures/` path prefix, suggesting `cyclonedx-valid.json` instead of `tests/fixtures/cyclonedx-valid.json`. The first two runs produced "no such file or directory" errors before the correct path was identified. GSD corrected itself each time, but the UAT loop is interactive for exactly this reason: the test descriptions define intent correctly even when the suggested command has an issue.

### Security Review

After UAT, GSD performed an automatic threat review against the Phase 1 implementation. Twelve threat categories were evaluated. All twelve were confirmed mitigated. One accepted risk was documented: the thin Cobra dependency boundary; the cobra command object is the only entry point for user-controlled input, which is appropriate for Phase 1 scope.

Commit: `23cade6 — docs(01): verify phase 1 threat mitigations`.

---

## Phase 1 Result: What Was Built

Starting from zero Go code, Phase 1 produced:

```
sbom-drift/
├── cmd/
│   ├── root.go          cobra root command
│   └── diff.go          diff subcommand
├── internal/
│   ├── model/
│   │   ├── sbom.go      SBOM, Component, Metadata, Dependency types
│   │   └── identity.go  purl-first matching logic
│   ├── parser/
│   │   ├── cyclonedxjson.go   CycloneDX JSON → internal model
│   │   ├── spdxjson.go        SPDX JSON → internal model
│   │   ├── detect.go          format autodetection
│   │   ├── parser.go          unified parser entry point
│   │   └── errors.go          fail-closed error taxonomy
│   ├── differ/
│   │   └── matcher.go   purl-first matching with fallback
│   └── app/
│       └── run.go       end-to-end execution path
├── tests/
│   └── fixtures/        CycloneDX and SPDX test SBOMs
└── main.go
```

And the planning artifacts that anchor the next session:

```
.planning/
├── PROJECT.md           what this is and why
├── REQUIREMENTS.md      21 testable requirements
├── ROADMAP.md           5-phase v1 plan
├── STATE.md             current progress
├── config.json          workflow settings
├── research/            domain research from 4 subagents
├── phases/
│   └── 01-input-normalization-foundation/
│       ├── 01-CONTEXT.md        locked decisions
│       ├── 01-DISCUSSION-LOG.md discussion history
│       ├── 01-01-PLAN.md        CLI and contracts
│       ├── 01-02-PLAN.md        parsers
│       ├── 01-03-PLAN.md        end-to-end wiring
│       ├── 01-UAT.md            5/5 tests passed
│       └── 01-SECURITY.md       12 threats verified
├── HANDOFF.json         machine-readable pause point
└── .continue-here.md    human-readable resume note
```

`go test ./...` passes. The CLI correctly parses CycloneDX JSON and SPDX JSON, matches components by purl, and fails closed on malformed or incomplete input. Phase 1 goal achieved.

---

## What I Actually Learned

### GSD vs Spec-Kit: Practical Differences

Having now used both frameworks for real projects, here is what the comparison looks like from the keyboard rather than a README:

| Aspect | GSD | Spec-Kit |
|---|---|---|
| Setup time | ~2 minutes via npx | ~5 minutes via CLI + constitution |
| First meaningful output | Questions start immediately | Constitution drafting first |
| Context rot handling | Explicit, built-in | Not the focus |
| Governance artifacts | Planning files in `.planning/` | Constitution in `.specify/`, specs in `specs/` |
| Session resumability | `HANDOFF.json` + `/gsd-resume-work` | Partial via memory files |
| UAT integration | Built into the workflow | Manual |
| Threat review | Automatic per phase | Manual |
| Branch strategy | Defaults to current branch | Creates a new branch per implementation spec |

The built-in UAT and threat review are the most practically useful differences. In the Spec-Kit session building apple-container-tui, I identified a parsing bug manually during testing. With GSD, the UAT sequence is designed into the workflow; you do not have to remember to test, and the results are committed as artifacts.

### The Value of Good Docs Before Coding

The pre-existing `docs/memory.md` and `docs/architecture.md` files paid off significantly during initialization. But those docs did not appear by accident.

Before writing a single line of code or touching GSD, I ran a dedicated planning session with GitHub Copilot in plan mode, using Claude Sonnet 4.6 with extended thinking enabled. The goal was not to generate code, but to pressure-test the design: the SBOM identity matching strategy, the offline-first constraint, the free-APIs-only policy, the output format decisions. High thinking mode forces the model to reason through trade-offs rather than surface the first plausible answer, and the output was a set of tightly scoped documents capturing decisions and their rationale.

That investment showed immediately when GSD ran its research subagents. They read `docs/memory.md` and `docs/architecture.md` alongside external documentation, which meant the generated requirements were specific rather than generic. GSD absorbed the purl-first matching strategy, the offline constraint, the free-APIs policy, and reflected all of it back into the 21 requirements without me having to repeat a word.

This is the actual value of the docs-first approach: not documentation for documentation's sake, but structured context produced through deliberate AI-assisted reasoning, which then makes every subsequent AI step dramatically more precise. The planning session was a multiplier on everything GSD did after it.

### Branch Strategy: A Lesson

One thing I noticed at the end of the session: GSD worked entirely on `main` by default. All commits went directly to the default branch without creating a feature branch or opening a pull request. For a solo project in early development this is acceptable, but it would be a problem for any team workflow requiring review gates.

GSD does support a `git.branching_strategy` setting (`none`, `phase`, or `milestone`) configurable via `/gsd-settings`. For future phases, I plan to configure `phase` branching so each implementation phase gets its own branch and merges via pull request. I will also update `AGENTS.md` with explicit instructions about branch usage so any AI agent working in this repo understands the expected workflow.

If you are adopting GSD for team projects, configure branching before the first execution phase.

### Context Window and `/clear`

GSD itself recommends running `/clear` between phases to reset the Copilot context window. This is context rot prevention in practice: do not carry the full conversation history from the previous phase into the next one. The planning artifacts in `.planning/` preserve the state; the context window does not need to.

I ran `/clear` between the discuss and plan phases, and the quality stayed consistent throughout. Make this a habit.

---

## Takeaways and Next Move

### Takeaways

GSD is a viable SDD framework. After one full session I can say it works, and it works without much ceremony: install takes two minutes, the first questions start immediately, and the implementation phase moves fast. The automated UAT and threat review built into the workflow are genuinely useful; you do not have to remember to test or to think about security surface area, both are part of the loop by default.

The YOLO mode mindset is real here. GSD is designed to push code, not to debate it. That is a deliberate trade-off and it shows.

My personal preference, especially for work contexts, is still Spec-Kit. The constitution system and the explicit governance artifacts give you a better audit trail, and the branch-per-spec workflow integrates more cleanly with team review processes. Spec-Kit asks more of you upfront; GSD asks less and moves faster. Both are legitimate depending on what you need.

That said, I am going to keep pushing sbom-drift with GSD. It earned that. And I plan to run the next phases inside a Docker Sandbox with `sbx`, which means full YOLO mode in an isolated environment: no credential leaks, no filesystem bleed, no network surprises. If you are not familiar with `sbx`, the [Docker Sandboxes article](/posts/docker-sandboxes-ai-agents/) covers the setup and the security model.

### Next Move

Phase 2 is "Core Drift and Deterministic Reporting": the actual diff results. To resume, the sequence is:

```
/gsd-resume-work
/gsd-discuss-phase 2
```

That is the phase where sbom-drift stops being a normalization foundation and starts being an actual drift detection tool. I will document it when it is done.

---

## References

- [GSD (Get Shit Done) — source and documentation](https://github.com/gsd-build/get-shit-done)
- [sbom-drift — project repository](https://github.com/matteobisi/sbom-drift)
- [Spec-Kit articles on this blog](/tags/spec-kit/)
- [Docker Sandboxes for AI agents (sbx)](/posts/docker-sandboxes-ai-agents/)

---
