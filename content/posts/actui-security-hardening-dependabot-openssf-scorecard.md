---
title: "Hardening ACTUI: Dependabot and OpenSSF Scorecard for a Side Project"
date: 2026-04-02T08:00:00Z
tags: [
  "devops", "devsecops", "security", "open-source",
  "github", "openssf", "dependabot", "apple-container",
  "spec-kit", "golang", "github-actions"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Back from KubeCon EU 2026 with a free Copilot Pro+ subscription, I turned my attention to the security posture of apple-container-tui. Here's how I added Dependabot and OpenSSF Scorecard using GitHub Actions, spec-kit, and the GitHub CLI."
canonicalURL: "https://www.msbiro.net/posts/actui-security-hardening-dependabot-openssf-scorecard/"
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
    alt: "ACTUI Security Hardening: Dependabot and OpenSSF Scorecard"
    caption: "Raising the security baseline of a solo side project"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## The Unexpected Swag from KubeCon EU 2026

[KubeCon EU 2026 Amsterdam](/posts/kubecon-eu-2026-amsterdam-recap/) was a great edition. I walked away with good conversations, new connections, and the usual conference bag full of stickers. But one thing stood out among the swag: **six months of GitHub Copilot Pro+**, courtesy of GitHub.

I'm not going to pretend I wasn't excited. Copilot Pro+ isn't cheap, and having it handed to you as conference loot—just because you showed up in the right place, accepting the right invitation—felt like a proper thank-you to the community. GitHub clearly knows its audience.

Once I got home and the post-conference wind-down was over, I had a renewed energy to work on my side projects. [apple-container-tui](https://github.com/matteobisi/apple-container-tui) (ACTUI) was the obvious choice. The tool had already gone through a few iterations: the [initial build with spec-kit](/posts/spec-kit-hands-on-apple-container-tui/) and a [follow-up with submenus and image management](/posts/actui-follow-up-team-usage-enhancements/). The code worked. The features were there. But the **repository security posture** was essentially zero.

That's what I decided to fix next.

## Why Security Automation Matters Even for Side Projects

Before jumping into the how, let me explain the why. I work as a security team leader. I'm genuinely interested in security practices, not just as a job function but as a mindset. And one thing that frustrates me is the common attitude that "it's just a side project, it doesn't need security."

It's a weak excuse for several reasons.

**Side projects often become something more.** ACTUI started as a proof-of-concept to test spec-kit workflows, but then I shared it with my team, wrote articles about it, and pushed other people to try it. The moment another person clones and runs your code, your security hygiene is their problem too.

**Automation is cheap to set up and runs forever.** Adding Dependabot and OpenSSF Scorecard to a repository is a one-time configuration effort. Once in place, you get continuous vulnerability scanning, dependency update PRs, and a public signal of how seriously you take supply chain security—without any ongoing manual work.

**It's a sandbox for learning real patterns.** Configuring these tools on a low-stakes project is the ideal way to understand them deeply before applying them in a professional context. Every misconfiguration you catch here is a lesson you don't have to learn the hard way on a production system.

## The Approach: Spec-Driven, as Always

Since ACTUI was already using spec-kit for all feature work, I kept the same discipline here. I created a new spec `006-repo-security-hardening` to capture the objectives before writing any YAML.

The main decisions locked into the spec before touching any files were clear: OpenSSF Scorecard must be a **required check before merge** (not just informational); Dependabot must cover both `gomod` and `github-actions` ecosystems on a monthly cadence; token permissions across all workflows must follow **least privilege**; and branch protection on `main` must enforce all of the above.

Starting from specification rather than from "let me just add a workflow file" is the right instinct. It forces you to think about what you actually want to enforce versus what you're adding out of cargo-cult habit. The spec becomes the reference when you're reviewing the PR later. With those decisions captured, implementation followed three concrete steps.

### Step 1: OpenSSF Scorecard

[OpenSSF Scorecard](https://securityscorecards.dev/) is an automated tool that evaluates open-source repositories against a set of security best practices. It checks things like branch protection, token permissions, pinned dependencies, code review requirements, CI presence, and more. Each check produces a score and a reasoning, and the aggregate gives you a public signal about the project's supply chain security posture.

Adding Scorecard to ACTUI meant creating `.github/workflows/scorecard.yml`. The workflow is straightforward: it runs on push to `main`, on schedule (weekly), and on pull requests. The output is published to the GitHub Security tab as SARIF.

The critical configuration detail is **workflow permissions**. Scorecard needs write access to the Security tab to publish results, but nothing else. The correct setup uses empty permissions at the workflow level and grants only what is needed at the job level:

```yaml
permissions: {}

jobs:
  analysis:
    permissions:
      security-events: write
      id-token: write
      contents: read
      actions: read
```

This is the **least-privilege pattern** for GitHub Actions workflows. Setting `permissions: {}` at the top level ensures that no job accidentally inherits broad defaults; each job must declare what it needs explicitly.

The other important detail: **pin all actions to immutable SHAs**, not floating tags. `ossf/scorecard-action@v2` is convenient but it's a moving target. If the tag is force-pushed to point at malicious code, your workflow runs that code with the permissions you granted. Using the full commit SHA prevents this:

```yaml
- uses: ossf/scorecard-action@f49aabe0b5af0936a0987cfb85d86b75731b0186  # v2.4.2
```

The SHA is pinned, the comment documents the human-readable tag. This is standard hygiene for any production-grade workflow.

### Step 2: Dependabot

Dependabot handles two distinct but related problems: **version updates** (keeping dependencies current) and **security updates** (patching known vulnerabilities automatically).

For ACTUI, the relevant dependency surfaces are Go modules and GitHub Actions. The `.github/dependabot.yml` configuration covers both:

```yaml
version: 2
updates:
  - package-ecosystem: "gomod"
    directory: "/"
    schedule:
      interval: "monthly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
```

Monthly cadence is the right default for a solo side project. Weekly would generate noise without adding meaningful security value; monthly keeps the dependency graph reasonably fresh without flooding your notification inbox.

One thing that's easy to miss: having `dependabot.yml` in the repository is **not the same** as enabling Dependabot security updates at the repository level. The YAML configures version update PRs. Security updates (automated fixes for CVEs affecting your dependency graph) require explicit enablement through the repository settings or via the GitHub API:

```bash
gh api \
  --method PUT \
  /repos/matteobisi/apple-container-tui/vulnerability-alerts

gh api \
  --method PUT \
  /repos/matteobisi/apple-container-tui/automated-security-fixes
```

After this, the full security automation stack was live: `dependabot_security_updates`, `secret_scanning`, and `secret_scanning_push_protection` all enabled.

### Step 3: Branch Protection

Repository files alone are not enforcement. A `scorecard.yml` workflow means nothing if someone can push directly to `main` and bypass it. Branch protection is what closes that gap.

I configured protection on `main` using `gh api` for repeatability. The key settings: **OSSF Scorecard as a required status check** (strict mode, meaning the branch must be up to date before merge); **PR review policy enabled**; **required conversation resolution**; force-push disabled; branch deletion disabled.

Required approvals are currently set to `0`. This is a pragmatic call for a solo-maintainer project: adding a `1` required approval when there is no second independent reviewer would just be friction with no security benefit. The moment there is a consistent second contributor, that moves to `1`.

```bash
gh api \
  --method PUT \
  /repos/matteobisi/apple-container-tui/branches/main/protection \
  --input branch-protection.json
```

Using `gh api` with an input file rather than clicking through the UI means the configuration is **auditable and repeatable**. If you ever need to recreate the repository settings from scratch, you have the exact payload documented.

## The Follow-Up: Workflow Hardening

After the initial PR landed, GitHub's security tooling flagged a broad token scope at the workflow level in the Scorecard workflow. This is precisely the kind of signal Scorecard is designed to surface, and it's the correct behavior: the tool found a real issue in the code it was reviewing.

The fix in PR #10 was clean: set `permissions: {}` at workflow level (not just job level), keep required write permissions only at job scope, and remove the `push` trigger from all branches (Scorecard only needs to run on `main` pushes). Small changes, but they close real attack surface on a workflow that has `security-events: write` access.

This is a lesson worth internalizing: **treat post-merge security findings as normal tuning, not failure**. The fact that Scorecard caught a permission issue in its own workflow is a feature, not an embarrassment. The process worked exactly as designed.

## Final Posture Snapshot

After both PRs, the ACTUI repository has a materially better security baseline:

- `main` is protected and cannot be pushed to directly
- `OSSF Scorecard` is a required check before any PR merges
- The Scorecard workflow runs with least-privilege, SHA-pinned actions
- Dependabot covers `gomod` and `github-actions` with monthly update PRs
- Dependabot security updates are enabled for CVE-driven fixes
- Secret scanning and push protection are active

The [OSSF Scorecard results](https://securityscorecards.dev/viewer/?uri=github.com/matteobisi/apple-container-tui) are publicly visible. That transparency is the point: anyone evaluating whether to use or contribute to the project can see the current security posture without having to audit the repository manually.

## What I Took Away from This

This iteration of ACTUI reinforced a few things I already believed but now have concrete evidence for.

**The spec-driven approach works even for operational tasks.** Writing a spec before adding Scorecard and Dependabot felt slightly over-engineered for such a small change, but it paid off: the clarification process forced me to decide upfront that Scorecard must be a *required* check, not just informational. Without that explicit decision, I might have defaulted to informational and ended up with a weaker enforcement posture.

**Post-merge security findings are information, not failure.** Scorecard surfaced a real permission issue in a workflow I had just written. That's the system working correctly. The right response is to fix and move on, not to question whether the tooling was worth adding.

**Copilot Pro+ made the iteration faster.** Having a capable AI assistant for the YAML plumbing, the `gh api` calls, and the workflow debugging reduced the friction enough that what could have been an evening of context-switching became a focused two-hour session. The conference swag turned out to be genuinely useful.

**Side projects are where you learn the patterns you'll use professionally.** Everything I practiced here (least-privilege workflows, SHA pinning, Dependabot scope tuning, branch protection via API), is directly applicable to production repositories. The stakes were low; the learning was real.

The complete project is at [github.com/matteobisi/apple-container-tui](https://github.com/matteobisi/apple-container-tui). The spec artifacts, PRs, and workflow configurations are all there if you want to see the full picture.

