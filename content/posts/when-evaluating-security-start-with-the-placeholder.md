---
title: "When Evaluating the Security of Your Project, Start With the Placeholder"
date: 2026-09-29
tags: [
  "cybersecurity", "devsecops", "supply-chain-security",
  "ai-agents", "mcp", "threat-modeling", "vulnerability-management"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "third-party.com was used as a placeholder in 1,700+ repos and became an attack surface. How to audit external references, mutable tags and trust assumptions."
keywords: [
  "placeholder domain security", "example.com vs third-party.com",
  "domain takeover risk", "security review checklist", "supply chain security",
  "mutable tag risk", "AI agent security", "MCP server security",
  "dependency trust assumptions", "security audit external references"
]
canonicalURL: "https://www.msbiro.net/posts/when-evaluating-security-start-with-the-placeholder/"
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
    alt: "A security review starting from seemingly harmless placeholders and external references"
    caption: "Security starts with understanding what a project trusts"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

We spend an enormous amount of time trying to secure software. We run SAST. We scan dependencies. We scan container images. We run IaC scanners. We check Kubernetes configurations. We monitor runtime behaviour. We build SBOMs. We integrate everything into CI/CD and create dashboards full of vulnerabilities, CVEs and security findings.

And yet sometimes the most interesting security problem is not a vulnerability in our software at all. Sometimes it is a **placeholder**.

A recent case reported by [The Hacker News](https://thehackernews.com/2026/09/placeholder-third-partycom-referenced.html) is a particularly good example. The domain `third-party.com`, which had been used as a generic documentation placeholder, was referenced across more than 1,700 public repositories.

There was only one problem. Unlike `example.com`, `example.org` and `example.net`, `third-party.com` was never reserved for documentation purposes. Someone could register it, and someone eventually did. The domain began serving malicious content.

| Timeline | Event |
|---|---|
| T0 | A developer writes `https://third-party.com` in a README as an example endpoint |
| T0 | The reference is copied into forks, templates, tutorials and agent skills |
| T1 | The original author leaves the project; nobody owns the string anymore |
| T2 | The domain is registered by an unrelated party |
| T3 | 1,700+ repositories now point at infrastructure nobody reviewed |

Nobody changed a line of code. The repositories were still "clean" the whole time.

---

## The Short Version

1. A domain that only **looks** like a placeholder is a dependency, not a comment.
2. The question is not "is this malicious right now?" but "**who controls this, and can that change?**"
3. Scanners classify strings. They do not understand **why** a string exists.
4. The same pattern applies to package registries, image tags, Git remotes and CI actions.
5. Documentation is code now, because agents read it and act on it.

> **If you don't control the domain, don't assume it is safe just because you use it as an example.**

---

## `example.com` Is Not the Same as `third-party.com`

There are domains specifically reserved for documentation and examples, defined in [RFC 2606](https://www.rfc-editor.org/rfc/rfc2606) and reinforced by [RFC 6761](https://www.rfc-editor.org/rfc/rfc6761). `example.com`, `example.org` and `example.net` exist so developers can use them without creating a dependency on somebody else's infrastructure.

Developers frequently use other domains because they look like placeholders:

```text
third-party.com
yourcompany.com
mycompany.com
your-api.com
vendor.com
acme.com
```

They look fictional. They look safe. A domain that merely looks like a placeholder is not a placeholder from a security perspective. The THN report describes several such domains used in repositories and agent skills, with some subsequently serving scams or malicious content.

| Domain | Reserved for examples | Registrable by third parties | Safe to reference |
|---|---|---|---|
| `example.com` | Yes | No | Yes |
| `example.org` | Yes | No | Yes |
| `example.net` | Yes | No | Yes |
| `example.test` | Yes (RFC 2606) | No | Yes |
| `localhost` | Reserved | No | Yes |
| `third-party.com` | No | Yes | **No** |
| `yourcompany.com` | No | Yes | **No** |
| `acme.com` | No | Yes | **No** |

If you find one of the unreserved names above in your repository, replace it with `example.com`, `example.org`, `example.net` or `example.test`.

---

## Scanners Check Reputation, They Do Not Check Ownership

When reviewing a project, we often ask:

> "Does this URL currently serve something malicious?"

That check is useful, but insufficient. A stronger review asks:

1. **Who controls this domain today?**
2. **Could somebody else control it tomorrow?**
3. **Why does our project reference it in the first place?**

Take this fragment:

```yaml
api:
  endpoint: https://third-party.com/api
```

A scanner resolves the URL, checks reputation, inspects the response and moves on. Today it returns nothing interesting. Tomorrow ownership changes.

| Asset | Changed? |
|---|---|
| The application | No |
| The deployment | No |
| The commit | No |
| The security properties of the application | **Yes** |

Nothing in the pipeline fires, because nothing in the pipeline watches ownership. A scanner can say `https://third-party.com` is a domain. It cannot say why the developer put it there:

| Possible intent | Typical signal in the repository |
|---|---|
| Documentation | Comment, README, tutorial, changelog |
| Unit test or fixture | `testdata/`, `tests/`, mocked or `localhost` target |
| Sample configuration | `.example` suffix, Helm values template, `.env.sample` |
| Real production endpoint | Referenced from runtime code, config loader, manifest |
| Dead reference | Untouched for years, resolves to a parked page |
| AI agent example | `skills/`, agent prompts, MCP configuration files |
| Copied template | The same string appears across many unrelated projects |

The same string is consumed by different systems, each with a different tolerance for a hostile value.

A string that looks harmless to a developer can become an instruction, or an actionable resource, for another system.

---

## AI Turns Documentation Into Infrastructure

The Hacker News report notes references to `third-party.com` in AI agent skills and MCP server documentation. Suppose an agent is given:

```text
Use the following endpoint for testing:

https://third-party.com
```

A human understands this is an example. An agent may treat it as an endpoint to fetch.

{{< mermaid >}}
graph TD
    subgraph BEFORE["Before: the human boundary"]
        D1["Developer reads the file"] -->|"understands intent"| C1["Nothing happens, the string is inert"]
    end

    subgraph AFTER["After: the agent boundary"]
        D2["Agent reads the file"] -->|"treats it as an endpoint"| F["Fetches the URL"]
        F --> R["Retrieved content is untrusted input"]
        R --> X["Command execution, API calls, file changes"]
    end

    D1 -.->|"same string"| D2
{{< /mermaid >}}

If the agent can browse the URL, call APIs or fold retrieved content into subsequent actions, the placeholder has become an input to an automated system. The boundary moved. I covered containment in [The Challenge of Securing AI Agents: A DevSecOps Perspective](/posts/securing-ai-agents-devsecops-challenge/); this is the input side.

Security reviews of AI-enabled projects need to cover code, dependencies, and the content agents are instructed to consume.

---

## Start With "What Do We Trust?"

Do not start only with "what vulnerabilities does this project have?". Start with "**what does this project trust, why does it trust it, and do we control it?**"

| We inspect | We implicitly assume |
|---|---|
| Dependencies and libraries | Nobody will ever transfer ownership |
| Container images | A mutable tag will always resolve to the same bits |
| Secrets and API endpoints | The hostname will keep pointing to the same service |
| Cloud resources and permissions | The account will not be deleted or resold |
| Network exposure | The firewall rule is still the one I wrote |
| Documentation and examples | Nothing in there is ever executed or fetched |

Most trust in a modern project sits outside the code you wrote. I detailed that distribution in [Supply Chain Attacks Won't Stop: 8 Controls to Reduce Your Exposure](/posts/supply-chain-attack-prevention-8-controls/). If your review stops at the CI pipeline, you are already late; I made that argument in [Shift Left Starts in the IDE](/posts/shift-left-starts-in-the-ide/).

The pattern repeats across surfaces:

### Domains and packages

For domains, search for `https://` and `http://`, then check each hit:

- Who owns it, is it reserved, and do we control it?
- Is it still required, and is it referenced from executable code or only documentation?
- For packages, add maintenance status, pinning and transitive dependencies.

Ownership transfer is the failure mode in both cases.

### Container images and Git remotes

```dockerfile
FROM some-image:latest
```

Who controls `some-image`, what does `latest` resolve to today, and who can overwrite it tomorrow? For a concrete remediation on the image side, see [In 2026 I Am Still Asked Why You Need a Hardened Container Image Catalog](/posts/hardened-images-catalog-2026-non-negotiable/).

```yaml
uses: somebody/action@main
```

carries a different assumption from:

```yaml
uses: somebody/action@<immutable-commit>
```

Check whether Git URLs, organisations, submodules and Actions still point where you expect, and pin mutable references to immutable commits.

### Documentation and forgotten references

Scan README files, examples, Helm values, tutorials and sample configurations, not just source code. Those files are increasingly inputs to automated systems (see [Securely Working with Third-Party MCP Servers](/posts/securely-using-third-party-mcp-servers/) if agents consume yours).

Forgotten references fail silently:

| Reference | How it got there | Why nobody notices |
|---|---|---|
| A URL in a README | Copied from a tutorial five years ago | It still returns HTTP 200 |
| A DNS record | Created for a dead project | The zone is still hosted |
| A CDN endpoint | Belongs to a discontinued service | The vendor never took it down |
| A cloud bucket | Created for a proof of concept | Nobody knows it is still there |

A broken dependency gets noticed. A working dependency under somebody else's control stays invisible. `third-party.com` survived long after its original assumption was forgotten.

---

## A Simple Exercise for Your Next Security Review

Do not start with the CVE database. Start with a search across the whole repository:

```bash
git grep -Eo 'https?://[^ ")]+' .
```

```bash
git grep -Ei 'curl |wget |http://|https://|npm |pip |docker pull|FROM '
```

Include `README.md`, `docs/`, `examples/`, `helm/`, `.github/`, `.gitlab/`, Dockerfiles, Terraform, Kubernetes manifests, MCP configurations, agent skills and sample configuration files.

Classify every external reference:

| Reference | Purpose | Owner | Mutable? | Required? |
|---|---|---|---|---|
| `example.com` | Documentation | Reserved | No | No |
| `internal-api.company.com` | Production API | Us | Controlled | Yes |
| `vendor.com` | Vendor API | Vendor | Yes | Yes |
| `random-domain.com` | Example | Unknown | Yes | No |

Remove first the rows that look like this:

```text
Owner: Unknown
Mutable: Yes
Required: No
```

---

## Conclusion

Tooling is excellent at finding known bad things (vulnerable packages, leaked credentials, misconfigured resources, suspicious behaviour). The interesting failures come from things never considered security-sensitive: a placeholder, a DNS name, a sample configuration, a forgotten CI action, a URL in a README. Something never supposed to matter, until somebody else controls it.

A vulnerability is a flaw in software you control, usually with a CVE and a patch. A trust assumption is a belief that something outside your control will keep behaving the same way. Trust assumptions fail silently, without a patch, and scanners do not track them.

Next time you evaluate a project, do not start with the scanner. Start with the placeholders. Ask what is real, what is reserved, what is controlled, what can change ownership, and what the application, the pipeline and the agents around it implicitly trust.

Sometimes the most dangerous dependency has no CVE. It is the one you never thought was a dependency at all.
