---
title: "Evaluating Oss Security Fresh Editor s2c2f"
date: 2025-12-27T16:37:11Z
tags: [
  "openssh", "supply-chain-security", "s2c2f", "fresh-editor", 
  "rust", "devsecops", "openssf", "scorecard", "semgrep", 
  "cargo-audit", "sbom", "oss-security", "kubernetes"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Holiday hacking from the couch: evaluating Fresh editor's security using OpenSSF Scorecard, Semgrep, and cargo audit. A practical guide to applying the S2C2F framework for secure OSS adoption without killing developer productivity. Learn how to vet unknown open-source tools in an afternoon before bringing them to corporate environments."
canonicalURL: "https://www.msbiro.net/posts/evaluating-oss-security-fresh-editor-s2c2f/"
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
    alt: "<alt text>"
    caption: "<text>"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

It's December 27th, and like most of you, I'm somewhere between "fully checked out for the holidays" and "can't stop tinkering with new tools on my laptop." Nobody's at work. Teams is shut down and Slack is quiet. The corporate VPN can wait until January. But my curiosity? That's working overtime.

A couple of weeks ago, I discovered **[Fresh](https://sinelaw.github.io/fresh/)**, a Rust-based terminal text editor that feels like it was designed specifically for people like me who live in terminals.  
Here's what caught my attention:

- **Out-of-the-box LSP support** – Autocomplete, go-to-definition, and diagnostics work immediately
- **Mouse support that actually works** – Click, drag, select. No keyboard-only masochism required
- **Standard keybindings** – Ctrl+S to save, Ctrl+F to find. No relearning muscle memory
- **Insane performance** – Opens a 2GB Kubernetes log file in 600ms using under 40MB of RAM
- **TypeScript plugins in a sandboxed Deno environment** – Extensible without sacrificing security
- **Zero configuration required** – No `.vimrc` archaeology, no Emacs Lisp spelunking. 

No configuration hell. No two-week learning curve. It just works.

I've been using it on my personal MacBook all week, and honestly? It's fantastic.  

But here's where my DevSecOps brain won't shut off: **Should I install this on my work laptop when I get back in January?** More importantly, **how do I even evaluate if it's safe?**

The maintainer, Noam Lewis (GitHub: `sinelaw`), is unknown to me. The project only gained public visibility in early December 2024. There's no corporate sponsor, no security audit, and definitely no compliance certification. But it solves real problems, and I know the second I mention it in our team chat, someone's going to ask: *"Did you vet this?"*

So instead of binge-watching another series, I decided to spend an afternoon doing what any reasonable security-minded person does during the holidays: **run some security checks from my couch.**

## **The Reality of OSS Adoption in 2025**

Let me paint you a picture. A developer discovers a brilliant CLI tool—maybe it's a better `grep`, a faster JSON parser, or in this case, a text editor that doesn't make them want to throw their laptop out the window. They install it locally, fall in love with it, and then quietly `brew install` it on their work machine without a second thought.

Meanwhile, security teams are trying to maintain SBOMs, track dependencies, and answer questions like *"Do we use anything with CVE-2025-XXXXX?"* while developers are operating in a parallel universe of shadow IT.

The problem isn't malicious intent—it's friction. Corporate OSS approval processes are often so painful that developers either wait weeks (killing productivity) or just bypass the process entirely (killing security). When I get back to work in January, I want to have a **fast, repeatable process** for evaluating tools like Fresh. Not a three-week ticket queue. Not a "trust me bro" vibe check. Something systematic.

That's where frameworks like [s2c2f](https://github.com/ossf/s2c2f) come in, but more on that in a moment. First, let me show you what I can actually test **right now**, on my personal MacBook, without any corporate infrastructure.

## **What I Can Check From My Couch: The Local Security Audit**

For these tests, I cloned the Fresh repository and checked out version **0.1.64**—the current release as of today.

### **Test 1: OpenSSF Scorecard**

The first thing I ran was the [OpenSSF Scorecard](https://scorecard.dev/), a free tool that evaluates open-source projects against security best practices. Here's what it gave me for Fresh:

```
scorecard --repo=github.com/sinelaw/fresh
Aggregate score: 4.5 / 10
```

**Ouch.** A 4.5 out of 10 isn't great. But let's dig into what that actually means:

**The Good News:**

- ✅ **Binary-Artifacts (10/10):** No pre-compiled binaries in the repo—everything's built from source
- ✅ **License (10/10):** GPL-2.0 license properly detected
- ✅ **Maintained (10/10):** 30 commits and 9 issues in the last 90 days—this project is actively maintained
- ✅ **Dependency-Update-Tool (10/10):** Dependabot or similar is configured
- ✅ **Dangerous-Workflow (10/10):** No risky GitHub Actions patterns
- ✅ **Vulnerabilities (8/10):** Only 2 existing vulnerabilities detected (likely minor)

**The Bad News:**

- ❌ **CI-Tests (0/10):** No CI tests run on merged PRs
- ❌ **Code-Review (0/10):** Only 1 out of 21 changesets had approval before merge
- ❌ **SAST (0/10):** No static analysis tools (like Clippy for Rust) running in CI
- ❌ **Signed-Releases (0/10):** No release signing or provenance
- ❌ **Security-Policy (0/10):** No SECURITY.md file for vulnerability reporting
- ❌ **Fuzzing (0/10):** No fuzzing infrastructure
- ❌ **Token-Permissions (0/10):** GitHub workflow tokens have excessive permissions
- ❌ **Pinned-Dependencies (0/10):** Dependencies not pinned by hash in CI

**My take:** This is a **young project** with good fundamentals (active maintenance, clean dependencies) but zero security maturity processes. It's a solo developer building something useful, not a corporation with a security team. That's not inherently bad—it's just reality. Most OSS starts here.

### **Test 2: SAST with Semgrep**

Since the scorecard showed no SAST tools running in CI, I decided to run one myself. [Semgrep](https://github.com/semgrep/semgrep) is a free static analysis tool that can catch security issues in code. Here's what I found:

```bash
semgrep --config=auto
Scanning 330 files...
┌─────────────────┐
│ 9 Code Findings │
└─────────────────┘
```

**The breakdown:**

- **6 findings in GitHub Actions workflows:** Potential command injection via `${{ inputs.version }}` without proper sanitization. This could allow an attacker with write access to inject malicious commands into the CI/CD pipeline.
- **1 finding in npm-package/binary-install.js:** Path traversal vulnerability in `path.join()` operations. An attacker could potentially access arbitrary files during npm package installation.
- **1 finding in plugins/git_log.ts:** Non-literal RegExp construction could allow ReDoS (Regular Expression Denial of Service) attacks if user input isn't sanitized.
- **1 finding in plugins/theme_editor.ts:** Prototype pollution vulnerability in object manipulation code.

**Reality check:** These are **potential** vulnerabilities, not confirmed exploits. The GitHub Actions issues only matter if someone with malicious intent gets write access to the repo (in which case, you have bigger problems). The plugin vulnerabilities are mitigated by Fresh's sandboxed Deno environment, which prevents plugins from arbitrary file access.

**What I'd want to see:** These findings triaged and fixed upstream, or documented as accepted risks with mitigations.

### **Test 3: Dependency Audit with cargo audit**

Next, I ran `cargo audit` to check for known vulnerabilities in Fresh's Rust dependencies:

```bash
cargo audit
Loaded 892 security advisories
Scanning Cargo.lock for vulnerabilities (492 crate dependencies)
warning: 2 allowed warnings found
```

**The findings:**

1. **paste 1.0.15:** Marked as unmaintained (RUSTSEC-2024-0436). This is a macro library used by `ratatui` (Fresh's TUI framework) and `deno_core` (plugin runtime).
2. **yaml-rust 0.4.5:** Marked as unmaintained (RUSTSEC-2024-0320). Used by `syntect` (Fresh's syntax highlighting library).

**Important context:** These are **warnings about unmaintained dependencies**, not active CVEs. The libraries still work fine—they're just not getting updates. The risk is that *future* vulnerabilities won't be patched. This explains the scorecard's 8/10 vulnerabilities score.

**What this means:** Fresh's direct dependencies are clean, but transitive dependencies (dependencies of dependencies) have some technical debt. This is common in the Rust ecosystem as projects migrate to newer alternatives. The immediate security risk is low, but it's something to monitor.

### **Test 4: Architecture and Language Safety**

Fresh is written in Rust, which gives us some built-in safety:
    
- **Memory-safe by default:** Rust eliminates entire classes of vulnerabilities (buffer overflows, use-after-free)
- **Sandboxed plugins:** The plugin system runs TypeScript in a **Deno environment**, which means plugins can't access files or network without explicit user permission. This is huge—it means even if someone publishes a malicious plugin, it's contained.[^1]


### **Test 5: Community Signals**

I looked at the community reception:

- 137+ upvotes on [Reddit](https://www.reddit.com/r/rust/comments/1pciab0/ann_fresh_a_terminalbased_editor_in_rusteasytouse/) with positive technical discussions
- [DistroTube](https://www.youtube.com/watch?v=dspEVA8eoUg) (a respected Linux YouTuber) gave it a favorable review
- Contributors from 2 different organizations (based on scorecard)

These aren't security guarantees, but they're **social proof** that people are paying attention and testing it.

## **The Framework I'd Use At Work: s2c2f**

Here's the thing: what I just did is fine for personal experimentation, but it's not enough for corporate adoption. When I get back to the office in January, I'd need to apply a proper framework and the best one I know for **consuming** OSS (not producing it) is [s2c2f](https://github.com/ossf/s2c2f).



The **Secure Supply Chain Consumption Framework** was developed internally at Microsoft and donated to OpenSSF in 2022. It's designed specifically for this problem: *"A developer found a cool tool. How do we evaluate if it's safe to use?"

S2C2F defines **eight practices** across **four maturity levels**:

### **The 8 Practices (Brief Overview):**

1. **Ingest It** – Mirror OSS to internal registries (survive registry outages, control what enters)
2. **Scan It** – Automated vulnerability scanning before developers can use it
3. **Inventory It** – Generate SBOMs so you know where every dependency lives
4. **Update It** – Patch faster than attackers exploit
5. **Enforce It** – Prevent direct pulls from public registries
6. **Audit It** – Verify compliance with approved ingestion paths
7. **Rebuild It** – Build from source on trusted infrastructure
8. **Fix It Upstream** – Fork and patch if maintainers are unresponsive

The genius of S2C2F is that it's **risk-based**. You don't need Level 4 maturity (rebuild everything from source) for a text editor. But for a secrets manager or a container runtime? Absolutely.

### **How I'd Apply S2C2F to Fresh in January:**

**Level 1 (Minimum Viable Security):**

- Mirror Fresh to our internal Artifactory with a 24-hour quarantine
- Run `cargo audit` and `semgrep` scans automatically during ingestion
- Generate an SBOM with `syft` and add it to our asset inventory
- **Time investment:** ~2 hours to set up, then automated

**Level 2 (For Broader Rollout):**

- Enable Dependabot alerts and auto-patching for Fresh's dependencies
- Implement a policy: developers can only install from internal registry
- Monthly audit of developer laptops to catch unapproved installations
- Require the 9 Semgrep findings to be triaged before approval
- **Time investment:** ~1 day for policy + tooling

**Level 3 (If Used in Production Workflows):**

- Rebuild Fresh from source on our CI/CD infrastructure
- Sign the binary with our corporate certificate
- Fork the repo and patch the unmaintained dependencies (`paste` and `yaml-rust`)
- **Time investment:** ~1 week for pipeline + process

For Fresh specifically, **Level 1-2 is appropriate**. It's a productivity tool, not a critical security component. The Semgrep findings and unmaintained dependencies are worth documenting, but they're not showstoppers for non-production use.

## **The Decision: What I'm Doing Right Now**

Here's my plan for the next few days:

**On my personal MacBook (now):**

- ✅ Keep using Fresh v0.1.64 for editing config files, logs, and personal projects
- ✅ Avoid using it for anything with secrets or production code
- ✅ Monitor the GitHub repo for updates and security issues

**When I get back to work (January):**

- 📋 Present the scorecard, Semgrep, and cargo audit results to the team
- 📋 Propose adding Fresh to our "approved tools" list with Level 1 S2C2F compliance
- 📋 File GitHub issues upstream about the Semgrep findings and unmaintained dependencies
- 📋 Set a 6-month review: if the project adds signed releases, improves scorecard score, or migrates off unmaintained deps, escalate to broader approval
- 📋 Share the evaluation process as a template for other "developer-discovered" tools

**What I won't do:**

- ❌ Install it on my work laptop until it's ingested through proper channels
- ❌ Use it for editing production code or secrets
- ❌ Recommend it for CI/CD pipelines without Level 3 compliance


## **Takeaways: Security Doesn't Have to Kill Innovation**

The 4.5/10 scorecard score isn't a dealbreaker: it's **context**. Fresh is a young, actively maintained project with good architectural choices (Rust + sandboxed plugins) but immature security processes. The Semgrep and cargo audit findings are typical for early-stage OSS. That's **fixable**.

The real lesson here is this: **you can evaluate OSS security in an afternoon.** OpenSSF Scorecard is free. Semgrep has a no-login tier. `cargo audit` is built into the Rust toolchain. The friction isn't technical: it's cultural!!  

Security teams need to build **fast approval paths** for low-risk tools, and developers need to respect that "I found it on Reddit" isn't a security strategy.

When I get back to the office, I'm going to propose we automate most of this. Any tool with a scorecard above 7/10, zero critical Semgrep findings, and a clean dependency scan should auto-approve for Level 1 use. Anything below that gets a manual review. Total turnaround: 48 hours, not 3 weeks.

Because here's the truth: Fresh is brilliant, and people like me will keep discovering tools like it.  

The question isn't **if** we adopt new OSS, it's **how fast we can do it safely**.


║  ⭐⭐⭐ See you in 2026 ! ⭐⭐⭐║         


```bash
{
  "message": "Happy Holidays!",
  "year": 2025,
  "author": "Matteo Bisi",
  "status": "enjoying_break" 
}
```