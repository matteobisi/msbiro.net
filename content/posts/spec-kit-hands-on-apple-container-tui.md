---
title: "Testing Spec-Kit: Building a Functional Container TUI in 2.5 Hours"
date: 2026-02-12T00:00:00Z
tags: [
  "devops", "devsecops", "AI", "spec-driven-development",
  "golang", "tui", "apple-container", "spec-kit", "open-source"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A hands-on journey building apple-container-tui from empty repository to working Go binary in 2.5 hours using spec-kit 0.1.0. Testing spec-driven development with a real POC."
canonicalURL: "https://www.msbiro.net/posts/spec-kit-hands-on-apple-container-tui/"
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
    alt: "Spec-Kit Hands-On: Building Apple Container TUI"
    caption: "From empty repo to working binary in 2.5 hours"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction: Theory Meets Practice

In my [previous article about GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/), I explored the theoretical foundations of spec-driven development: why structured AI workflows matter for compliance, auditability, and team collaboration. I discussed the high-level concepts of audit trails, liability, and how spec-kit transforms "vibe coding" into a rigorous, documented process.

Today, I'm sharing something different: a raw, unfiltered hands-on experience building a real tool from scratch using spec-kit. This is a chronological journey documenting what actually happened when I let spec-kit drive the development process from constitution to working code.

**The Challenge**: Build `apple-container-tui`, a Terminal User Interface (TUI) for Apple Container that eliminates the need to memorize complex CLI syntax. Think of it as a guided, interactive wrapper around macOS 15.2+'s native container runtime.

**The Constraint**: Use spec-kit (CLI Version 0.1.0, Template Version 0.0.94, released 2026-02-11) to drive the entire development process from empty repository to working binary.

**The Result**: A fully functional Go TUI in 2.5 hours, complete with all planned features, proper error handling, and a polished user experience.

The complete project is available on GitHub: [matteobisi/apple-container-tui](https://github.com/matteobisi/apple-container-tui)

---

## The Setup: Starting from Zero

### 21:25 - Initial Repository Creation

The journey began with a fresh repository and a clear goal: create a tool that makes Apple Container accessible to users who don't want to memorize command syntax. Having recently worked with Apple Container for my previous article on markitdown integration, I knew the pain points firsthand.

First step: initialize spec-kit in the repository.

```bash
gh repo create apple-container-tui --public --clone
cd apple-container-tui
spec-kit init
```

The `spec-kit init` command sets up the foundational structure. Unlike traditional development where you might immediately start coding, spec-kit enforces a discipline: define your project's constitution first. This document becomes the DNA of your project, the immutable principles that guide all subsequent decisions.

### 21:27 - Crafting the Constitution

The constitution is stored in `.specify/memory/constitution.md` and serves as the project's north star. For apple-container-tui, I needed to capture several critical aspects: **Purpose** (simplify Apple Container operations through an intuitive TUI); **Target Users** (macOS 15.2+ users who want container functionality without CLI complexity); **Quality Standards** (robust error handling, clear user feedback, production-ready code); **Constraints** (terminal-native, local-only operation, keyboard navigation).

Notably, I was intentionally vague about the implementation language at this stage. I wanted to test how spec-kit would identify the optimal technology through its planning process rather than prescribing it upfront.

You can view the [complete constitution here](https://github.com/matteobisi/apple-container-tui/blob/main/.specify/memory/constitution.md).

The constitution isn't just documentation—it becomes the context that spec-kit uses to generate specifications, validate implementations, and maintain consistency across the entire development lifecycle. Every decision, every generated spec, every code suggestion gets filtered through this constitutional lens.

### 21:30 - First Specification Attempt

With the constitution in place, I moved to specification generation:

```bash
spec-kit spec
```

This command launches an interactive session where you describe *what* you want to build. Spec-kit, armed with your constitutional principles, transforms your natural language requirements into structured specifications.

My initial input focused on the core functionality:

> "Create a TUI for Apple Container that allows users to list running containers, pull images, run containers with interactive selection of parameters, and manage container lifecycle—all without memorizing command syntax."

The first generated spec was comprehensive but needed refinement. One of spec-kit's strengths is its iterative nature; you're not locked into the first draft. I requested adjustments to emphasize error handling patterns, clarify the container parameter selection workflow, and ensure the TUI remained lightweight without unnecessary features.

### 21:35 - Plan Phase: Technology Selection

After the initial specification, spec-kit entered the plan phase—a critical step where it analyzes requirements and determines the optimal technology stack. I had been deliberately vague about implementation details in the constitution; this was intentional to test how the AI agent would navigate the technology selection process.

The plan phase launched a sub-agent that evaluated multiple options: Python with textual, Rust with ratatui, Go with bubbletea. The decision was documented in the [plan.md file](https://github.com/matteobisi/apple-container-tui/blob/main/specs/001-apple-container-tui/plan.md):

> **Language/Version**: Go 1.21+ (chosen for optimal balance of productivity, performance, binary distribution, and TUI library maturity)

The reasoning was sound: Go's single-binary distribution model simplifies deployment on macOS, the bubbletea TUI framework is mature and well-documented, and Go's standard library provides robust command execution capabilities for wrapping the Apple Container CLI. The decision wasn't mine; it emerged from the spec-driven process evaluating the technical requirements against the constitutional constraints.

This demonstrates one of spec-kit's key strengths—it can guide technical decisions when you provide clear requirements but remain open to implementation approaches.

---

## The Development Cycle: Iteration by Design

### 21:45 - Task Generation and Implementation

With an approved specification, spec-kit generated a task list—the implementation roadmap. This isn't a generic TODO list; each task is derived directly from the specification, maintaining traceability from requirement to implementation.

View the [generated task list](https://github.com/matteobisi/apple-container-tui/blob/main/specs/001-apple-container-tui/tasks.md).

The tasks were organized into logical phases with 100 total tasks across 6 phases: project scaffolding and dependency management, core bubbletea TUI framework implementation, Apple Container integration layer, interactive workflows for each operation, error handling and user feedback, testing and validation.

Before proceeding to implementation, I ran a project analysis for consistency, which identified 3 critical, 5 high, 8 medium, and 5 low issues. The critical issues included command naming inconsistency (contracts used `container` but data-model referenced `acrun list`) and missing CLI checker logic. Spec-kit proposed concrete remediation edits for all critical and high-priority issues, which I approved and applied.

### 22:30 - Implementation Phase

After resolving the consistency issues, I launched the implementation phase with `/speckit.implement`. The AI agent systematically worked through all 100 tasks, creating the project scaffolding, Go module dependencies, foundational models and services, bubbletea app shell and key bindings, and contract tests. The entire implementation phase completed autonomously, with the agent checking off each task as it progressed through the phases.

### 23:40 - Testing and Bug Fix

After building the binary, I tested the TUI and discovered a parsing bug: the program expected the old `container list --all` headers (`CONTAINER ID`, `STATUS`, `PORTS`), but the latest Apple Container CLI now returns different headers (`ID`, `STATE`, `ADDR`). This caused the error "missing header CONTAINER ID" and an empty container list.

I reported the issue to spec-kit, which updated the parser to support both header formats by adding a fields-based fallback for the newer header style, added test coverage for the new format, and verified the fix with `go test ./src/services`. After rebuilding, the TUI correctly listed running containers.

### 23:56 - Success: Working Binary

After 2.5 hours of development, I had a working binary that lists running containers with formatted output, pulls images with progress feedback, runs containers with interactive parameter selection, stops containers with confirmation prompts, handles errors gracefully with actionable messages, and maintains clean code structure aligned with Go conventions.

The final test: running a container end-to-end without touching the `applecontainer` CLI directly. It worked flawlessly.

---

## The Technical Journey: What Actually Happened

The reality? The process was almost entirely automatic. After defining the constitution and providing the initial spec-kit prompt with my requirements, the AI agent handled the heavy lifting with minimal intervention.

The specification was generated in one go, capturing all the essential features: container listing, image pulling, running containers with interactive parameter selection, and lifecycle management. The clarification phase asked me a few questions (keyboard vs mouse support, error handling when CLI not installed, manual vs automatic refresh), then I requested one clarification about which programming language would be used; the AI explained that decision comes in the planning phase, not the spec phase.

The plan phase selected Go with bubbletea based on the technical constraints. Before implementation, I ran a consistency analysis that caught 21 issues (3 critical, 5 high, 8 medium, 5 low) across the planning artifacts, command naming inconsistencies, missing CLI checker logic, unclear config paths. I approved the proposed fixes, and the implementation proceeded smoothly through all 100 generated tasks.

The only manual intervention during coding was fixing a parsing bug after testing: the program expected old `container list --all` headers (`CONTAINER ID`, `STATUS`, `PORTS`), but the latest Apple Container CLI changed to different headers (`ID`, `STATE`, `ADDR`). I reported the issue, spec-kit updated the parser to handle both formats and added test coverage.

![apple-container-tui in action](apple-container-tui.png)

That's it. Two clarifications during planning (language selection timing, consistency fixes), one bug fix during testing (header parsing). Everything else—dependency resolution, error handling patterns, state management architecture, testing validation criteria—was orchestrated by spec-kit autonomously.

This was the promised value of spec-driven development realized: you define *what* you want through a constitution and high-level requirements, then let the structured workflow handle the *how* with surgical precision.

---

## What I Actually Learned

This wasn't my first attempt at spec-driven development—I covered the theory in my [previous article about GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/). Here's what 2.5 hours of real development taught me.

**The structure accelerated development.** The constitution-spec-task workflow had a learning curve in the first hour, but once the rhythm clicked, it became smooth. When the spec is clear, you're not cycling between "what should this do?" and "how do I code this?"—the spec handles the first, the AI agent handles the second.

**Consistency analysis prevented bugs.** Running project analysis after task generation identified 21 issues (3 critical, 5 high, 8 medium, 5 low) before any code was written. Command naming inconsistencies, missing CLI checker logic, unclear config paths—all caught during planning. Preventative quality assurance, not reactive debugging.

**Technology decisions emerged from requirements.** Being deliberately vague about implementation language was an experiment. The plan phase sub-agent evaluated Python/textual, Rust/ratatui, and Go/bubbletea against requirements, then documented the reasoning for Go 1.21+. The constitutional constraints and technical requirements drove the decision, not personal preference.

**The artifact trail is the real value.** The automatically generated audit trail—every architectural decision, every iteration, every spec refinement—makes this approach transformative for compliance-focused teams. Someone can read the [constitution](https://github.com/matteobisi/apple-container-tui/blob/main/.specify/memory/constitution.md), [plan](https://github.com/matteobisi/apple-container-tui/blob/main/specs/001-apple-container-tui/plan.md), and [task list](https://github.com/matteobisi/apple-container-tui/blob/main/specs/001-apple-container-tui/tasks.md) and understand the entire project without reading a line of code. Knowledge transfer becomes spec reading, not code archaeology.

**The only bug was environmental.** The header parsing issue wasn't a flaw in the spec-driven process—it was an upstream change in Apple Container's CLI output format. Spec-kit adapted quickly, updating the parser to handle both formats and adding test coverage.

If you're considering spec-driven development, try it on a small project first. The workflow feels unnatural initially, writing constitutions before code violates the "just start coding" instinct—but the systematic refinement and automatic documentation are worth it. Beyond GitHub's Spec-Kit, [OpenSpec](https://github.com/Fission-AI/OpenSpec) offers a viable open-source alternative with similar artifact-driven workflows.

The complete project is available at [github.com/matteobisi/apple-container-tui](https://github.com/matteobisi/apple-container-tui).  
The specs, the iterations, the decisions—all documented, all traceable, all real.

---
