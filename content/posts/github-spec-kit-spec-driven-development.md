---
title: "GitHub Spec-Kit: Why Structured AI Development Beats Vibe Coding"
date: 2026-01-21T09:23:27Z
tags: [
  "devops", "devsecops", "AI", "spec-driven-development",
  "open-source", "compliance", "github", "spec-kit"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A DevSecOps team leader's perspective on GitHub Spec-Kit, spec-driven development, and why structured AI workflows matter for compliance, auditability, and team collaboration."
canonicalURL: "https://www.msbiro.net/posts/github-spec-kit-spec-driven-development/"
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
    alt: "GitHub Spec-Kit for DevSecOps"
    caption: "Bringing structure to AI-assisted development"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction: Spec-Driven Development vs. Vibe Coding

If you've been working with AI coding assistants, you've probably experienced what some call "vibe coding", throwing prompts at an LLM and hoping for the best. Sometimes it works brilliantly. Other times, you end up with code that technically runs but doesn't align with what you actually needed, or worse, introduces architectural decisions that create technical debt down the road.

Spec-Driven Development (SDD) flips this approach on its head. Instead of starting with code and documenting later (if at all), you begin with comprehensive specifications that define the *what* and *why* before anyone, human or AI, writes a single line of code. The specification becomes the single source of truth, guiding implementation and ensuring alignment across the entire team.

For DevSecOps teams, this shift is significant. We're not just shipping features; we're responsible for security, compliance, and maintainability. When AI-generated code goes straight into production without proper documentation, we lose the audit trail. We lose the reasoning behind decisions. We lose the ability for team members to pick up where others left off.

## What is GitHub Spec-Kit?

[GitHub Spec-Kit](https://github.com/github/spec-kit) is an open-source toolkit that brings structure and rigor to AI-assisted software development. Released by GitHub, it provides templates, a CLI, and integration points with AI coding agents like GitHub Copilot, Claude, and Cursor.

The toolkit implements a four-phase workflow:

1. **Specify**: Define the high-level specification focusing on user outcomes, goals, and success criteria, without diving into technical implementation details.

2. **Plan**: Translate the specification into a technical architecture, outlining components, technologies, and constraints.

3. **Tasks**: Break down the architecture into actionable work items with clear descriptions and acceptance criteria.

4. **Implement**: Generate code that's validated against both the spec and the plan, with feedback loops for continuous refinement.

Here's a visual representation of the Spec-Kit workflow:

```
┌────────────────────┐
│ Specify            │
│ (define outcomes)  │
└────────┬───────────┘
         │ share intent with your AI
         ▼
┌────────────────────┐
│ Plan & Review      │
│ (architecture)     │◀──── feedback loop ──────┐
└────────┬───────────┘                          │
         │ approved plan                        │
         ▼                                      │
┌────────────────────┐                          │
│ Tasks & Implement  │──────────────────────────┘
│ (AI writes code)   │
└────────┬───────────┘
         │ validated code
         ▼
┌────────────────────┐
│ Document & Ship    │
│ (versioned specs)  │
└────────────────────┘
```
What makes Spec-Kit interesting is its "constitution" system; you can define team standards and process requirements that the AI must follow. This governance layer is particularly valuable for teams with strict compliance needs.

For more details, check out:
- [GitHub Spec-Kit Repository](https://github.com/github/spec-kit)
- [Official Documentation](https://github.github.com/spec-kit/)
- [GitHub Blog Announcement](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [Den Delimarsky youtube channel](https://www.youtube.com/@DenDev/videos/)

## Why I Like Spec-Kit for My DevSecOps Team

After experimenting with Spec-Kit, I've found several aspects particularly valuable for how we work:

### Forces Deeper Reasoning

The "Specify" phase isn't just documentation, it's a forcing function. When engineers have to articulate the *what* and *why* before implementation, they think through edge cases, security implications, and integration points they might otherwise miss. The AI doesn't just accept your spec; it challenges you, asks clarifying questions, and often surfaces considerations you hadn't thought of. I've seen this back-and-forth generate ideas that would have been missed in a typical "just build it" prompt.

### Built-in Documentation and Decision History

Here's what sold me: the markdown files generated by Spec-Kit live in your repository. They're versioned. They're auditable. For teams dealing with NIS2 compliance, ISO 27001, or any regulatory framework requiring documented decision-making processes, this is huge. You're not scrambling to reconstruct why something was built a certain way six months later, the reasoning is right there in the commit history.

### Team Continuity

When someone leaves the team or takes over a repository they didn't build, the spec files provide context that code comments never capture. The architectural decisions, the trade-offs considered, the requirements that shaped the implementation, it's all documented in a structured format that any team member can follow.

### Compliance and Audit Readiness

For regulated industries, being able to demonstrate that AI-assisted development follows a documented, reviewable process matters. Spec-Kit creates artifacts that auditors understand: requirements, plans, and implementation records that show deliberate decision-making rather than "the AI suggested this."

These structured markdown files live in your repository and **establish a full chain of accountability** from the initial specification through planning, task breakdown, code generation, to final review.  

Pull requests and merge requests become fully traceable; every change links back to its originating spec, with commit history preserving who approved what and why. This audit trail satisfies frameworks like NIS2, ISO 27001, or SOC 2, proving human oversight over AI outputs while maintaining dev-to-approver traceability.

## Comparison with Similar toolkit

Spec-Kit isn't the only player in this space. Two alternatives worth mentioning:

### OpenSpec

[OpenSpec](https://github.com/Fission-AI/OpenSpec/) is a lightweight, open-source spec-driven framework that excels at managing changes in existing codebases. Its delta-based approach (tracking ADDED, MODIFIED, REMOVED requirements) makes it particularly suited for brownfield projects. If you're working with legacy systems rather than greenfield development, OpenSpec's minimal footprint and focus on change management might be a better fit. It's also tool-agnostic, working across Claude, Cursor, Copilot, and others without requiring specific integrations.

### BMAD Method

[BMAD (Breakthrough Method for Agile AI-Driven Development)](https://github.com/bmad-code-org/BMAD-METHOD) takes a more comprehensive approach with 19 specialized AI agents handling different phases of development. It's powerful for enterprise-scale projects requiring rigorous documentation and traceability, but the complexity can be overkill for smaller teams or iterative changes. If you need full lifecycle governance with specialized agents for product management, architecture, and QA, BMAD delivers, but expect a steeper learning curve.

### Quick Comparison

| Aspect | Spec-Kit | OpenSpec | BMAD |
|--------|----------|----------|------|
| Best for | New projects, structured teams | Legacy/brownfield projects | Large enterprise projects |
| Learning curve | Moderate | Low | High |
| Governance level | High | Moderate | Very High |
| Setup complexity | Moderate | Minimal | Complex |

For DevSecOps teams starting with spec-driven development, I'd recommend Spec-Kit or OpenSpec; these frameworks are similar in promoting structured AI workflows but have key differences that guide the choice: 
- Spec-Kit for greenfield projects building from scratch, thanks to its comprehensive four-phase process (Specify → Plan → Tasks → Implement) and constitution system that establishes clean governance without legacy constraints; 
- OpenSpec for existing codebases needing incremental changes, with its lightweight specs/changes folder model that handles brownfield refactoring, feature diffs, and maintenance naturally. 
- BMAD serves a different purpose, best for enterprise-scale projects requiring multi-agent complexity across full development lifecycles with specialized agents for product management, architecture, and QA.

## Key Takeaways

Let's be honest: AI is everywhere in software development now. If someone tells you they're not using it at all, they're either working in an extremely regulated environment or they're not being entirely truthful. The question isn't whether to use AI; it's how to use it responsibly.

**Use AI ethically and correctly**: this means respecting customer data privacy, being transparent about AI-assisted development where required, and maintaining human oversight of critical decisions. Spec-driven development helps here by creating accountability; every AI-generated artifact traces back to a human-approved specification.

**Structure accelerates, not constrains**: it might seem counterintuitive, but adding process to AI-assisted development actually speeds things up. Less rework, fewer misunderstandings, better alignment between what you asked for and what you get. The time invested in specifications pays dividends in reduced debugging and refactoring.

**Documentation is not optional**: for DevSecOps teams like mine, audit trails matter. Whether it's NIS2, SOC 2, or internal compliance requirements, being able to demonstrate how and why code was written (including AI-assisted code) is increasingly non-negotiable.

**Pick the right tool for your context**:  spec-Kit is excellent for teams wanting structured governance in new projects. OpenSpec shines for incremental changes in existing systems. BMAD handles enterprise-scale complexity. Match the tool to your actual needs.

The AI hype cycle will continue, but thriving teams should integrate these tools thoughtfully, capturing AI speed while preserving documentation, auditability, and collaboration.