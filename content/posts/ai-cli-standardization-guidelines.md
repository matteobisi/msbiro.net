---
title: "AI CLI Standardization: Why Context Files Matter More Than Models"
date: 2026-02-06T11:38:56Z
tags: [
  "AI", "devops", "devsecops", "CLI", "standardization",
  "open-source", "best-practices", "AGENTS.md"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A practical guide to standardizing AI CLI workflows with context files, AGENTS.md, and environment management tools. Learn how to make your AI setup portable across tools and teams."
canonicalURL: "https://www.msbiro.net/posts/ai-cli-standardization-guidelines/"
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
    alt: "AI CLI Standardization and Guidelines"
    caption: "Making AI tools portable and predictable"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction: From Web Chatbots to CLI Tools

AI is a powerful tool, and for IT professionals, the most effective way to leverage it is through CLI tools like [GitHub Copilot CLI](https://github.com/github/gh-copilot), [Claude Code](https://www.anthropic.com/claude), [Gemini CLI](https://github.com/google-gemini/gemini-cli), or similar agents. In previous articles like [GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/), I explored spec-driven development and structured AI workflows, but I realized I skipped fundamental concepts: **why CLI tools beat web chatbots** and **how to standardize your AI setup for portability**.

The AI landscape evolves at the speed of light. Today, Model A delivers the best results; tomorrow, a different provider offers better cost/performance. Working through web interfaces locks you into platforms and creates painful migrations. CLI-based workflows, combined with standardized context files, make your AI setup portable, reproducible, and future-proof.

---

## Why CLI Tools Beat Web Chatbots

Web-based AI chatbots (ChatGPT, Claude Web, Gemini Advanced) are convenient for quick questions, but they fall short for serious software development. CLI tools dominate for four critical reasons:

**Direct File System Integration**: CLI tools operate natively in your environment, reading and writing files directly, navigating repositories, executing commands, and applying changes across multiple files atomically. Web chatbots force constant copy-paste friction that destroys productivity and introduces transcription errors.

**Context Files as Ground Rules**: This is the game-changer. CLI tools automatically read project-specific context files (`AGENTS.md`, `CLAUDE.md`, `gemini.md`) that establish how AI should operate in your repository. These files define coding standards, security boundaries, build commands, architectural decisions, and project-specific vocabulary. The AI consumes these files automatically, ensuring consistent behavior without repeating instructions. Web chatbots have no equivalent; you'd paste the same context repeatedly.

**Portability Across Providers**: When you structure workflows around standardized context files rather than provider-specific interfaces, migrating between tools becomes trivial:

```bash
# Today: GitHub Copilot CLI
gh copilot suggest "implement user authentication"

# Tomorrow: Claude Code
claude "implement user authentication"

# Next week: Gemini CLI
gemini "implement user authentication"
```

The AI tool changes, but your context files (`AGENTS.md`, build scripts, documentation) remain constant. Each tool reads the same ground rules and produces aligned results.

**Version Control and Team Collaboration**: Context files live in your repository, versioned with Git, reviewed in pull requests, shared across teams. When developers join or AI tools update, everyone (human and AI) works from the same documented expectations. Web chatbot conversations are ephemeral and isolated to individual users.

---

## The AGENTS.md Standard and Context Organization

[AGENTS.md](https://agents.md/) has emerged as the unified convention for AI-specific context, replacing the proliferation of tool-specific files. Based on analysis of over 2,500 repositories, the community converged on this single file as the source of truth for AI coding agents.

### Why AGENTS.md Works

- **Cross-tool compatibility**: Works with GitHub Copilot, Claude, Cursor, Gemini CLI, and others
- **Discovery mechanism**: Tools search from current directory up to repository root, then user home directory
- **Precedence rules**: More specific (closer) files override general ones, enabling nested configurations in monorepos
- **Separation of concerns**: Keeps AI-specific instructions out of README.md, reducing clutter for human contributors

### What Belongs in AGENTS.md

Based on best practices from thousands of projects, effective AGENTS.md files cover:

```markdown
# Build & Test
- `npm run build` — compiles TypeScript to dist/
- `npm test` — runs Jest test suite
- `docker-compose up` — starts local dev environment

# Architecture Overview
React frontend (/frontend), Node.js API (/api), background workers (/workers).
Data flow: client → API → PostgreSQL.

# Code Conventions
- TypeScript strict mode
- Prefer async/await over callbacks
- All database mutations through DataService abstraction
- Follow airbnb-typescript eslint rules

# Git Workflow
- Branch naming: feat/*, fix/*, chore/*
- All PRs require one reviewer approval
- Conventional Commits format

# Security Boundaries
- Never modify /vendor or /secrets
- Never commit .env files or API keys
- Use environment variables for all configuration
- Test user input for injection vulnerabilities

# Project Vocabulary
- "Users" = individuals; "Accounts" = organizations
- "Sessions" are Redis-backed with 24-hour expiry
- Background jobs use BullMQ with Redis

# Useful Links
- [API Documentation](https://docs.example.com/api)
- [Architecture Diagram](https://docs.example.com/arch)
```

**Key principles**: Keep it under 200-300 lines, lead with actionable commands (build, test, lint), explicitly mark forbidden actions, use realistic examples instead of vague descriptions, and update when architectural decisions change.

### Organizing Advanced Context with ./agents Folder

For complex projects, organize specialized skills and commands in an `./agents` folder:

```
project-root/
├── AGENTS.md              # Main context file
├── agents/
│   ├── skills/
│   │   ├── kubernetes.md  # K8s-specific operations
│   │   ├── database.md    # Database migration patterns
│   │   └── security.md    # Security scanning workflows
│   ├── commands/
│   │   ├── deploy.sh      # Custom deployment scripts
│   │   └── analyze.py     # Code analysis utilities
│   └── templates/
│       ├── api-endpoint.md
│       └── component.md
```

This modular approach works for:
- **Specialized Skills**: Domain-specific knowledge (GraphQL API patterns, Kubernetes deployments, database migrations)
- **Custom Commands**: Project-specific automation scripts AI can invoke or reference
- **Templates**: Reusable code patterns for common structures (API endpoints, React components, test files)

Reference the folder in your AGENTS.md:

```markdown
# Additional Context
For specialized operations, see ./agents/skills/:
- Kubernetes: ./agents/skills/kubernetes.md
- Databases: ./agents/skills/database.md
- Security: ./agents/skills/security.md
```

**Note**: Some AI CLI tools like GitHub Copilot CLI (version 0.0.401+, released January 2026) automatically load the `.agents/skills` directory, eliminating the need for manual references in AGENTS.md. Check your tool's documentation for auto-discovery capabilities.

### Claude Code Workaround

While most modern AI CLI tools support AGENTS.md natively, **Claude Code** (as of early 2026) requires a pointer file. Create a `CLAUDE.md` that references AGENTS.md:

```markdown
# CLAUDE.md
This project uses the AGENTS.md standard for AI context.

Please read and follow all instructions in:
- ./AGENTS.md (primary AI context)
- ./agents/skills/ (specialized operation guides)

All guidelines, security boundaries, and coding conventions 
are documented in AGENTS.md.
```

This hybrid approach maintains a single source of truth while ensuring compatibility. When Claude Code adds native support, simply delete the pointer file.

---

## Security and Governance with AGENTS.md

For organizations adopting AI CLI tools at scale, **security and governance** are critical. The AGENTS.md specification includes enterprise features designed for DevSecOps environments, audit trails, and compliance requirements.

### Signed Manifests for Integrity

AGENTS.md supports **cryptographic signatures** to verify context file authenticity. This prevents tampering and ensures AI tools operate only with approved instructions:

```bash
# Sign AGENTS.md with GPG
gpg --clearsign --default-key your-key@example.com AGENTS.md

# Produces AGENTS.md.asc with embedded signature
```

Enterprise AI CLI tools (GitHub Copilot Enterprise, Claude for Business) can validate signatures before loading context:

```yaml
# .agents/config.yml (example enterprise config)
security:
  require_signed_manifests: true
  trusted_keys:
    - fingerprint: "0x1234567890ABCDEF"
      owner: "security-team@example.com"
  reject_unsigned: true
```

**Use cases**: 
- **Supply chain security**: Prevent malicious context injection in open-source dependencies
- **Compliance requirements**: SOC 2, ISO 27001 environments requiring documented AI behavior
- **Multi-tenant platforms**: Ensure customers' AI agents can't access each other's contexts

### Audit Trails and Telemetry

Enterprise AI tooling provides **telemetry hooks** to log AI interactions for security review:

```toml
# mise.toml with audit logging
[env]
AI_AUDIT_LOG = "/var/log/ai-agents/audit.log"
AI_TELEMETRY_ENDPOINT = "https://telemetry.example.com/v1/events"

[tasks.ai-audit-check]
run = """
echo "Checking AI audit logs..."
jq 'select(.action == "file_write")' $AI_AUDIT_LOG | tail -20
"""
```

**Audit events** typically capture:
- Context files loaded (AGENTS.md, skill modules)
- Files accessed, created, or modified by AI
- Commands executed by AI agents
- Security boundary violations (forbidden file access)
- API calls to external services

Example audit log entry:

```json
{
  "timestamp": "2026-02-03T18:15:42Z",
  "tool": "github-copilot-cli",
  "version": "0.0.401",
  "user": "dev@example.com",
  "action": "context_loaded",
  "files": ["AGENTS.md", "agents/skills/kubernetes.md"],
  "checksum": "sha256:a1b2c3...",
  "signature_valid": true
}
```

### Security Boundaries in Practice

Reference security frameworks directly in AGENTS.md for automatic enforcement:

```markdown
# Security Boundaries (SOC 2 Compliance)

## Forbidden Operations
- Never read or write files in /secrets, /vendor, /.env
- Never execute commands with sudo or privilege escalation
- Never commit API keys, tokens, or credentials
- Never access production databases from local environment

## Required Validations
- All user input must pass parameterized query validation
- File uploads checked against MIME type whitelist
- Authentication required for all API endpoints except /health

## Audit Requirements
- Log all database mutations to audit_log table
- Record AI-generated code changes in git commits with "AI-Assisted:" prefix
- Weekly security scans: `npm audit && trivy scan .`

## External Service Access
Allowed domains for AI API calls:
- api.openai.com (GPT models)
- api.anthropic.com (Claude models)
- generativelanguage.googleapis.com (Gemini models)

Block all other outbound connections during AI sessions.
```

AI tools respecting AGENTS.md spec will refuse operations violating these boundaries and log violations to audit trails.

### Governance at Scale

For organizations with multiple teams and repositories:

**Centralized policy repository**:
```
corporate-ai-policies/
├── AGENTS.md (base template)
├── agents/
│   ├── skills/
│   │   ├── corporate-security.md
│   │   ├── compliance-checks.md
│   │   └── approved-dependencies.md
│   └── policies/
│       ├── data-classification.md
│       └── third-party-integrations.md
└── mise.toml (approved tool versions)
```

Teams inherit base policies via Git submodules or package distribution, customizing for project-specific needs while maintaining corporate security baselines.

**Continuous validation** in CI/CD:

```yaml
# .github/workflows/validate-ai-context.yml
name: Validate AI Context
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Verify AGENTS.md signature
        run: gpg --verify AGENTS.md.asc
      - name: Check forbidden patterns
        run: |
          # Ensure AI context doesn't allow dangerous operations
          ! grep -i "sudo\|rm -rf\|DROP DATABASE" AGENTS.md
      - name: Validate against schema
        run: npx agents-md-validator AGENTS.md
```

This ensures AI context files meet security standards before deployment, preventing misconfigurations that could expose systems to AI-assisted attacks.

---

## Environment Management with mise.toml

Beyond code context, AI workflows need **reproducible environment setup**. [`mise`](https://mise.jdx.dev/) (formerly rtx, rebranded in 2024) is a universal tool version manager ensuring everyone (developers, CI/CD, AI agents) runs identical tool versions.

AI CLI tools depend on specific language runtimes (Python, Node.js), SDK versions, or supporting utilities. Without version locking, you get "works on my machine" problems where AI-generated code runs locally but fails in CI or for teammates.

**Why mise over alternatives**: mise is backwards-compatible with [asdf](https://asdf-vm.com/) (the previous ecosystem standard), supporting its plugin ecosystem while delivering 20-100x faster performance through Rust implementation. If you're migrating from asdf, mise reads `.tool-versions` files natively, making adoption seamless.

**Example mise.toml for AI workflows**:

```toml
[tools]
python = "3.11.7"
node = "20.11.0"
golang = "1.22.0"
# AI-specific CLI tools
"npm:@anthropic/claude-cli" = "1.5.0"
"npm:gh-copilot" = "1.0.3"

[env]
_.file = ".env"
_.file = ".ai-env"  # AI API keys, separate from app config

[tasks.ai-check]
run = """
echo "Verifying AI toolchain..."
claude --version
gh copilot --version
python --version
"""

[tasks.ai-test]
run = """
npm test
python -m pytest tests/
"""
```

**Benefits**: Deterministic installations (`mise install` gives identical versions), automatic activation (entering directory activates environment), CI/CD integration (GitHub Actions uses `mise-action` to replicate local setup), and task definitions for standardized workflows.

**CI/CD Integration**:

```yaml
# .github/workflows/ai-test.yml
name: AI-Assisted Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: jdx/mise-action@v2
      - run: mise install
      - run: mise run ai-test
```

This ensures AI toolchain versions match between local development, CI/CD, and teammates' machines.

### Advanced mise Features for AI Workflows

**Version Scoping**: mise supports hierarchical version configuration, crucial for monorepos or nested projects:

```bash
# Global defaults in ~/.config/mise/config.toml
[tools]
python = "3.11"
node = "20"

# Project override in ~/my-project/mise.toml
[tools]
python = "3.12"  # Override global Python

# Nested service in ~/my-project/services/api/mise.toml
[tools]
node = "21.5.0"  # API needs newer Node
```

When AI tools operate in `~/my-project/services/api`, they get Node 21.5.0 and Python 3.12 (inherited from parent). The closest `mise.toml` wins, enabling precise control in complex repositories.

**Backend Aliases for Stability**: Pin tools to specific backends to avoid breaking changes:

```toml
[tools]
# Use core:python instead of python plugin for stability
"core:python" = "3.11.7"

# Pin npm packages with version ranges
"npm:@anthropic/claude-cli" = "^1.5.0"  # Compatible updates
"npm:gh-copilot" = "=1.0.3"             # Exact version only
```

**Environment Variable Templates**: Construct dynamic environments for AI tools:

```toml
[env]
_.file = ".env"
PROJECT_ROOT = "{{ config_root }}"
PYTHON_BIN = "{{ python_path }}"
AI_MODEL_CACHE = "{{ env.HOME }}/.cache/ai-models"
```

AI CLI tools can reference `$PROJECT_ROOT` in scripts, ensuring portability across team members' machines.

**Task Dependencies and Hooks**: Automate setup steps before AI operations:

```toml
[tasks.setup]
run = "pip install -r requirements.txt && npm install"

[tasks.ai-generate]
depends = ["setup"]
run = """
gh copilot suggest "implement new feature"
"""

[hooks.after]
# Run linter after AI generates code
"tasks:ai-generate" = "npm run lint"
```

This ensures AI-generated code passes quality checks automatically.

**asdf Migration Path**: If your project uses asdf's `.tool-versions`:

```bash
# mise reads .tool-versions automatically
cat .tool-versions
# python 3.11.7
# nodejs 20.11.0

mise install  # Installs from .tool-versions

# Optionally convert to mise.toml for advanced features
mise migrate .tool-versions mise.toml
```

No manual conversion needed; mise maintains compatibility with the asdf ecosystem (1000+ plugins).

---

## Complementary Context Strategies

Beyond AGENTS.md and mise.toml, these patterns strengthen AI workflows:

**Specification Files**: As covered in my [Spec-Kit article](/posts/github-spec-kit-spec-driven-development/), structured specs (Spec-Kit, OpenSpec) provide high-level context about features, requirements, and architectural decisions. They complement AGENTS.md by focusing on *what* and *why* rather than *how*.

**Architecture Decision Records (ADRs)**: Document key technical choices with rationale in `docs/adr/`:

```markdown
# ADR-001: Use PostgreSQL for Primary Database

## Context
We need persistent storage for user accounts and transactional data.

## Decision
Use PostgreSQL 15 with TimescaleDB extension for time-series metrics.

## Consequences
- Requires Docker Compose for local dev
- Enables SQL-based analytics without separate warehouse
- Team needs PostgreSQL expertise for optimization
```

Reference ADRs in AGENTS.md so AI understands why certain patterns exist.

**Configuration Templates**: Provide `.env.example` so AI understands required environment variables:

```bash
# .env.example
DATABASE_URL=postgresql://localhost/myapp
REDIS_URL=redis://localhost:6379
OPENAI_API_KEY=sk-...
LOG_LEVEL=info
```

**Pre-commit Hooks**: Configure automated checks AI-generated code must pass:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.0
    hooks:
      - id: black
```

Reference these in AGENTS.md so AI generates code that passes linting.

**Workflow Automation**: Encapsulate complex workflows in Makefiles or scripts:

```makefile
# Makefile
.PHONY: setup test deploy

setup:
	mise install
	npm install
	docker-compose up -d

test:
	npm run lint
	npm test
	python -m pytest

deploy:
	./scripts/deploy.sh production
```

Include these in AGENTS.md's "Build & Test" section.

---

## Practical Setup: Implementing Standardized AI Context

Here's how to implement these patterns in a new or existing project:

**Step 1: Create AGENTS.md**

```bash
cd your-project
cat > AGENTS.md << 'EOF'
# Build & Test
- `npm install` — install dependencies
- `npm run build` — compile TypeScript
- `npm test` — run test suite

# Architecture
Microservices: API (Node.js), Workers (Python), Frontend (React).

# Security
- Never commit .env files
- All secrets via environment variables

# Git Workflow
- Branch: feat/*, fix/*, chore/*
- PRs require 1 reviewer

# Conventions
- TypeScript strict mode
- ESLint airbnb-typescript
- Prefer async/await

# Forbidden
- Never modify /vendor or /secrets
EOF
```

**Step 2: Add Claude Code workaround (if needed)**

```bash
cat > CLAUDE.md << 'EOF'
# CLAUDE.md
This project uses the AGENTS.md standard.

Please read and follow: ./AGENTS.md
EOF
```

**Step 3: Create ./agents structure (optional)**

```bash
mkdir -p agents/{skills,commands,templates}

cat > agents/skills/deployment.md << 'EOF'
# Deployment Skill

## Production
- `make deploy-prod` — requires AWS credentials
- Always run tests first

## Staging
- `make deploy-staging`
EOF
```

**Step 4: Set up mise.toml (if needed)**

```bash
cat > mise.toml << 'EOF'
[tools]
node = "20.11.0"
python = "3.11.7"

[env]
_.file = ".env"

[tasks.test]
run = "npm test && python -m pytest"
EOF

mise install
```

**Step 5: Add configuration template**

```bash
cat > .env.example << 'EOF'
DATABASE_URL=postgresql://localhost/db
REDIS_URL=redis://localhost:6379
API_KEY=your-key-here
EOF
```

**Step 6: Document in README**

```markdown
## AI-Assisted Development

This project uses AI CLI tools with standardized context:
- Read AGENTS.md for AI-specific guidelines
- Claude Code users: CLAUDE.md points to AGENTS.md
- Run `mise install` to set up correct tool versions
- Copy `.env.example` to `.env` and configure
```

**Step 7: Commit and share**

```bash
git add AGENTS.md CLAUDE.md agents/ mise.toml .env.example README.md
git commit --signoff -m "feat: add AI context standardization"
git push
```

Now anyone using AI CLI tools in this repository gets consistent, predictable behavior aligned with your project's standards.

---

## Key Takeaways

**CLI tools beat web chatbots for development**: Direct file access, persistent context files, and reproducible environments make terminal-based AI workflows vastly more productive than browser-based interfaces.

**Standardization enables portability**: When you structure context around universal files like AGENTS.md rather than tool-specific interfaces, migrating between AI providers becomes trivial. The AI landscape changes fast; your workflow shouldn't break when new models emerge.

**Context files are executable instructions**: AGENTS.md and similar files aren't just reference material. They're ground rules AI tools consume to maintain consistency across thousands of interactions without repeated manual prompting.

**Environment determinism matters**: Tools like mise ensure AI-generated code works identically across local development, teammates' machines, and CI/CD. Version drift destroys AI assistance value when code works locally but fails elsewhere.

**Start simple, grow as needed**: Begin with basic AGENTS.md covering build commands and security boundaries. Add mise.toml when version drift causes problems. Introduce specs and ADRs when compliance or team scale demands it. Don't over-engineer day one.

---

## Tool Support and Evolving Standards

The AI CLI tooling ecosystem evolves **rapidly**. This section reflects the state as of **February 2026**; always verify current support with official tool documentation.

### Feature Support Matrix

| Feature | GitHub Copilot CLI | Claude Code | Gemini CLI | Cursor Editor |
|---------|-------------------|-------------|------------|---------------|
| AGENTS.md native support | ✅ v0.0.401+ | ❌ (Not available) | ✅ v1.2.0+ | ✅ v0.40+ |
| .agents/skills auto-load | ✅ v0.0.401+ | ❌ (Not available)| ⚠️ Experimental | ⚠️ Uses SKILL.md |
| Signed manifest validation | ✅ Enterprise | ❌ (Not available)| ❌ (Not available)| ⚠️ Beta |
| mise.toml integration | ⚠️ Manual | ⚠️ Manual | ✅ Native | ⚠️ Manual |
| Audit logging | ✅ Enterprise | ✅ Enterprise | ❌ (Not available)| ❌ (Not available)|

**Legend**: ✅ Fully supported | ⚠️ Partial/Beta | ❌ Not available
**Note**: I don't use Cursor, I've provided a more extensive comparison based on web research
### Staying Current

Verify tool versions and feature support regularly:

```bash
# Check current versions
gh copilot --version
claude --version
gemini --version

# Search for feature announcements
gh copilot changelog | grep -i "agents.md"
```

**Monitor the specification**: The [AGENTS.md spec](https://agents.md/) maintains a living document of tool support and emerging extensions (security policies, telemetry schemas).

**Expect convergence**: Major providers (GitHub, Anthropic, Google) are collaborating through the [AI Context Working Group](https://aicontext.org/). By late 2026, expect universal support across all major CLI tools, standardized security schemas, and certified compliance templates for SOC 2 and ISO 27001.

**Future-proofing**: Structure your AI context around universal patterns (build commands, security boundaries, architectural docs) rather than tool-specific features. Review your AGENTS.md and mise.toml configurations **quarterly** to eliminate workarounds as tools mature.

---

## References

- [AGENTS.md Official Specification](https://agents.md/)
- [GitHub: How to Write a Great AGENTS.md](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
- [Mise Documentation](https://mise.jdx.dev/)
- [My Previous Article: GitHub Spec-Kit](/posts/github-spec-kit-spec-driven-development/)
- [OpenAI: Custom Instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md)

The AI ecosystem will continue evolving, but well-structured context and standardized workflows give you flexibility to adapt without rewriting your development process. Build portability into your AI setup today, and you'll be ready when the next generation of models arrives tomorrow.
