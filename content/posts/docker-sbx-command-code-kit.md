---
title: "Docker Sandboxes Kits: Running the Command Code TUI Inside a microVM"
date: 2026-11-18T06:00:00Z
tags: ["container-security", "ai-security", "devsecops", "ai-agents", "docker-sandboxes"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Docker Sandboxes ships templates for a catalogue of coding agents, but Command Code is not one of them. I built a v3 kit from scratch, installed the cmd TUI in a Node image, injected the API key through the host proxy, and ran it inside a microVM."
canonicalURL: "https://www.msbiro.net/posts/docker-sbx-command-code-kit/"
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
    alt: "Docker Sandboxes kit for the Command Code TUI"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---


In April I wrote about [Docker Sandboxes](/posts/docker-sandboxes-ai-agents/): `sbx` runs a coding agent in YOLO mode inside a microVM, and a host proxy injects its credentials so the agent never sees them. The built-in roster is long: Claude, Codex, Copilot, Cursor, Devin, Docker Agent, Droid, Kiro, OpenCode, and Shell.

At work I use Claude, which is on that list. For personal projects I pay for [Command Code](https://commandcode.ai), a low-cost subscription that gives a lot for the price. Its TUI, `cmd`, is not on the list.

`sbx` supports custom kits, so this post builds one for Command Code. It covers the descriptor, the Dockerfile, credentials, the network policy, and a real session inside the sandbox. The code is in [docker-sbx-command-code-kit](https://github.com/matteobisi/docker-sbx-command-code-kit).

---

## Kits in one minute

A kit is an OCI image with a YAML descriptor attached. There are two kinds:

- A **workload** supplies the base environment and the command to run. You pass it to `sbx run` or `sbx create`. A new agent is a workload.
- A **mixin** adds tools to an existing workload through `--kit`.

The Dockerfile says what goes into the image. The descriptor declares *capabilities*: typed requests the host fulfils around the image, such as network rules, credentials, and agent instructions.

v3 kits need `sbx` v0.45 or later.

## What Command Code needs

Two things:

- **Node 22 or later.** The `command-code` npm package declares `engines.node >= 22`, but npm doesn't enforce it by default, so pick the base image accordingly.
- **One API endpoint.** The CLI reads `COMMAND_CODE_API_KEY` and sends it as `Authorization: Bearer` to `api.commandcode.ai`. I confirmed the host by reading the CLI bundle and by calling `/alpha/whoami` with the key.

Ripgrep isn't on the list because the package bundles its own copy through the `@vscode/ripgrep` dependency.

One host and one header make Command Code an easy agent to sandbox. The kit also ships `gh`. It isn't part of the agent: it's the worked example of adding a second tool with its own rules.

---

## The kit

The kit is two files. Their names must match before the extension, because the build finds the Dockerfile from the descriptor's filename:

```text
command-code/
├── command-code.yaml
└── command-code.dockerfile
```

### The descriptor

```yaml
# syntax=docker/sandbox-kit:3
schemaVersion: "3"
displayName: Command Code (cmd)
author: "Matteo Bisi"
description: >-
  The Command Code TUI (cmd) on Node.js 22, with the GitHub CLI as a second
  command, proxy-managed credentials, scoped egress, and a sandbox AGENTS.md.
sourceUrl: https://github.com/matteobisi/docker-sbx-command-code-kit

kind: workload

args:
  version:
    default: "latest"
    pattern: '^(latest|[0-9]+\.[0-9]+\.[0-9]+)$'
    description: Command Code release to install (latest, or x.y.z to pin)
    buildArg: COMMAND_CODE_VERSION
  ghVersion:
    default: "2.103.3"
    pattern: '^[0-9]+\.[0-9]+\.[0-9]+$'
    description: GitHub CLI release to install
    buildArg: GH_VERSION

capabilities:
  - type: com.docker.sandbox/sbx@1

  - type: com.docker.sandbox/network-policy@1
    config:
      runtime:
        allow:
          - api.commandcode.ai:443
          - api.github.com:443
          - github.com:443

  - type: com.docker.sandbox/credential@1
    description: Command Code API access
    config:
      service: commandcode
      phase: runtime
      apiKey:
        name: COMMAND_CODE_API_KEY
        proxyManaged: true
        inject:
          - domain: api.commandcode.ai
            header: Authorization
            format: "Bearer %s"

  - type: com.docker.sandbox/credential@1
    optional: true
    description: GitHub API access for gh
    config:
      service: github
      phase: runtime
      apiKey:
        name: GH_TOKEN
        proxyManaged: true
        inject:
          - domain: api.github.com
            header: Authorization
            format: "Bearer %s"
          - domain: github.com
            header: Authorization
            format: "Bearer %s"

  - type: com.docker.sandbox/agent-context@1
    config:
      filename: AGENTS.md
      directory: /home/agent/.commandcode
      content: |
        # Command Code sandbox

        You are running inside a Docker Sandboxes microVM, not on the
        developer's laptop.
        # ...credentials are placeholders, a 403 is the policy working,
        # persist env vars in /etc/sandbox-persistent.sh (full text in the repo)
```

Each capability does one job:

- **`sbx@1`** tells `sbx` that the image is ready for it to manage the agent's launch. The host starts the agent under bash as the image's user, and the entrypoint never becomes PID 1.
- **`network-policy@1`** lists three hosts: one for the agent and two for `gh`, with no wildcards. Every domain a credential is injected into must also appear here, or the build fails.
- **The first `credential@1`** declares `commandcode`, a service that `sbx` doesn't ship. That is supported: `sbx secret set commandcode` stores a value under the same name, with no registration step.
- **The second `credential@1`** reuses the built-in `github` service and is optional.
- **`agent-context@1`** writes an `AGENTS.md` into `~/.commandcode/`, which Command Code loads as user-level memory. It tells the agent not to "fix" the placeholder credential and not to work around a 403.

### The Dockerfile

```dockerfile
FROM node:22-bookworm-slim

ARG COMMAND_CODE_VERSION
ARG GH_VERSION
ARG TARGETARCH

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ca-certificates curl git jq less procps ripgrep unzip \
 && rm -rf /var/lib/apt/lists/*

# Additional command: GitHub CLI, pinned and checksum-verified. TARGETARCH
# (amd64 or arm64) matches the release asset names directly.
RUN asset="gh_${GH_VERSION}_linux_${TARGETARCH}.tar.gz" \
 && base="https://github.com/cli/cli/releases/download/v${GH_VERSION}" \
 && curl -fsSLo /tmp/gh.tgz "${base}/${asset}" \
 && echo "$(curl -fsSL "${base}/gh_${GH_VERSION}_checksums.txt" | grep " ${asset}\$" | cut -d' ' -f1)  /tmp/gh.tgz" \
      | sha256sum -c - \
 && tar -xzf /tmp/gh.tgz -C /usr/local --strip-components=1 --wildcards '*/bin/gh' \
 && rm /tmp/gh.tgz \
 && gh --version

# Installed as root into /usr/local, so cmd is on PATH and the agent cannot
# modify its own binary. The cache mount keeps npm tarballs out of the image.
RUN --mount=type=cache,target=/root/.npm \
    npm i -g --no-fund --no-audit "command-code@${COMMAND_CODE_VERSION}" \
 && cmd --version

# The image ships uid 1000 as `node`; the sandbox runtime expects `agent`.
RUN usermod --login agent --home /home/agent --move-home node \
 && groupmod --new-name agent node \
 && mkdir -p /home/agent/workspace /home/agent/.commandcode \
 && chown -R agent:agent /home/agent \
 && install -o agent -g agent -m 0644 /dev/null /etc/sandbox-persistent.sh \
 && echo '. /etc/sandbox-persistent.sh' > /etc/profile.d/sandbox-persistent.sh \
 && echo '. /etc/sandbox-persistent.sh' >> /etc/bash.bashrc

# Every HTTPS request leaves through a TLS-terminating proxy whose CA sbx adds
# to the system store; Node reads only its own bundle unless told otherwise.
ENV BASH_ENV=/etc/sandbox-persistent.sh \
    NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt \
    NODE_OPTIONS=--disable-warning=UNDICI-EHPA \
    DO_NOT_TRACK=1

USER agent
WORKDIR /home/agent/workspace

ENTRYPOINT ["cmd", "--yolo", "--trust", "--skip-onboarding", "--no-auto-update"]
CMD []
```

What matters here:

- **The platform floor.** The image needs bash, curl, git, a CA store, and a non-root user named `agent` with uid 1000 and home `/home/agent`. The Node image already uses uid 1000 for a user called `node`, so the Dockerfile renames that user instead of adding a second one.
- **The agent can't change its own binary.** `npm i -g` runs as root, so `cmd` lands in `/usr/local/bin` and is owned by root. Auto-update is off and the npm registry isn't in the allow list, so the agent can't replace it either. The cache mount keeps npm's tarballs out of the image.
- **Pinned extras.** `gh` is a pinned release with a verified checksum. Adding another tool takes three edits: a pinned install here, its hosts in the network policy, and a credential entry if it needs one.
- **Environment.** `NODE_EXTRA_CA_CERTS` makes Node trust the proxy's CA. `NODE_OPTIONS` hides a warning Node prints above the TUI whenever proxy variables are set. `DO_NOT_TRACK=1` turns off the CLI's telemetry exporters, which the policy would block anyway.
- **The entrypoint.** `--yolo` turns off permission prompts, `--trust` skips the project trust question, `--skip-onboarding` skips the first-run prompt, and `--no-auto-update` keeps the installed version. The sandbox is the boundary, so the agent gets the latitude.

---

## Build and run

Building a local kit goes through the host Docker API, so keep a Docker engine running while you iterate. The sandboxes themselves don't need Docker Desktop. Start by checking what `sbx` makes of the kit:

```bash
sbx kit validate ./command-code
sbx kit inspect ./command-code
```

```text
  Name:           command-code
  ...
  Schema:         v3
  Binary:         cmd
  AI File:        /home/agent/.commandcode/AGENTS.md
  Run Options:    --yolo --trust --skip-onboarding --no-auto-update
  Policies:
    Network:      3 allow, 0 deny
    Credentials:  2 sources
    Environment:  0 variables, 2 proxy-managed
```

That output is the whole contract: three hosts, two proxy-managed credentials, one command, one instruction file. The `kit` subcommands and `--kit-arg` are still marked experimental in the CLI reference.

Store the API key from `commandcode.ai/studio` on the host, then create the sandbox and launch it:

```bash
sbx secret set commandcode           # prompts, stores it in the OS keychain
sbx secret set github                # optional, for gh

sbx create --name command-code ./command-code ~/projects/hello-cmd
sbx run --name command-code
```

A relative kit path must start with `./`, because a bare name means an agent or a sandbox. That is why `./command-code` is the kit and `command-code` is the sandbox.

The kit is third party, so the first create asks you to approve its credentials:

```text
This kit wants to use these credentials:
  commandcode:  API key sent to api.commandcode.ai (stored)

Approve these credentials?
  1. Approve all
  2. Review each
  3. No   start without them
Choice [1-3, default 3]:
```

Watch the default. Pressing Enter picks `3`, which starts the sandbox without the credential. The first model call then fails with an error that looks like a network problem. Choosing `1` records the approval in `~/.config/sbx/credentials.yaml`. That file holds the service and its domains, never the key. With `--detach` or in CI nobody can answer the prompt, so write the binding there in advance.

### Using it on your own project

This is where I tripped up first. From my projects folder, `sbx run docker-sbx-command-code-kit/command-code msbiro.net` failed with `401 unauthorized` from Docker Hub. `sbx` only treats a kit reference as local when it is an explicit path, starting with `./`, `../`, `/` or `~/`. Anything else is a sandbox name, a built-in agent, or an image to pull.

The other trap is the name. A sandbox is bound to the workspace it was created with, so `--name command-code` reopens the demo sandbox and ignores the new path. Give each project its own sandbox:

```bash
cd ~/GitRepo/Personal
sbx run --name msbiro-net ./docker-sbx-command-code-kit/command-code ./msbiro.net
sbx run --name msbiro-net            # later, from any directory
```

A shell alias saves typing the kit path:

```bash
alias sbx-cmd='sbx run ~/GitRepo/Personal/docker-sbx-command-code-kit/command-code'
sbx-cmd --name msbiro-net ~/GitRepo/Personal/msbiro.net
```

---

## The TUI inside the sandbox

Here is a real session on a small Node repository. I asked for a `--shout` flag, a README update, and a verification run:

```text
 EDIT  [README.md]
 └  Updated README.md with 7 additions
 [...]
 SHELL  [node greet.js --shout World]
 └ HELLO WORLD

⠶ Done. greet.js now parses --shout and uppercases the greeting, the README documents
  it, and node greet.js --shout World outputs HELLO WORLD.

✻ Worked for 11s
────────────────────────────────────────────────────────────────────────────────────────
  » permission bypass on [shift+tab]
```

`permission bypass on` comes from the entrypoint flag: the agent wrote files and ran a shell command without a single prompt. The workspace is bind-mounted, so the changes are already on the host:

```bash
git diff --stat
#  README.md | 7 +++++++
#  greet.js  | 7 +++++--
```

Review that diff the way you would review a pull request from a contractor. The bind mount also gives the agent access to Git hooks, Makefiles and CI files, and changes there are easy to miss. To keep the host repository out of reach entirely, `sbx create --clone` runs the agent on a clone inside the container, and you fetch its commits from the `sandbox-<name>` remote.

If you want permission prompts back, skip `sbx run` and start the TUI yourself. `sbx exec` runs the binary directly and ignores the entrypoint flags:

```bash
sbx exec -it command-code cmd --permission-mode default
```

---

## Verifying the boundary

Five checks show what the agent can and cannot reach:

```bash
sbx exec command-code uname -r
# 7.0.14                       a separate kernel

sbx exec command-code cat /proc/1/cgroup
# 0::/docker/16d24907...       a container inside the VM's own engine

sbx exec command-code ls /Users/matteo.bisi/
# GitRepo                      only the path to the workspace exists

sbx exec command-code sh -c 'command -v docker || echo "not installed"'
# not installed                no socket, no CLI; dockerd@1 would add a private engine

sbx exec command-code sh -c 'echo $COMMAND_CODE_API_KEY'
# proxy-managed
```

The last check is the credential. The CLI sends `Bearer proxy-managed`, and the host proxy swaps in the real key from the keychain on the way to `api.commandcode.ai`. The key never exists inside the VM, so the agent can't read it, log it, or commit it. A headless call confirms the swap works:

```bash
sbx exec command-code cmd -p "Reply with exactly: proxy auth ok"
# proxy auth ok
```

By default a sandbox gets every host CPU and half the host memory: 10 vCPUs and 16 GiB on this 32 GB laptop. `--cpus` and `--memory` change that.

---

## Network policy, in practice

Kit rules are added to your own policy, and a deny from any source wins. To test this, I denied the GitHub hosts and the npm registry for this sandbox:

```bash
sbx policy deny network --sandbox command-code github.com
sbx policy deny network --sandbox command-code api.github.com
sbx policy deny network --sandbox command-code registry.npmjs.org
```

`gh` stopped working, and `curl` to both GitHub hosts returned 403. `cmd -p` still answered, because the agent needs only `api.commandcode.ai`. `sbx policy log command-code` shows the split:

```text
BLOCKED REQUESTS
HOST                     PROXY    REASON
api.github.com:443       forward  Denied by local rule
github.com:443           forward  Denied by local rule
registry.npmjs.org:443   forward  Denied by local rule

ALLOWED REQUESTS
HOST                      PROXY
api.commandcode.ai:443    forward
```

### The governance trap

On my first attempt every request returned 403, including requests to `api.commandcode.ai`. My Docker account belonged to an organization with centralized sandbox governance. Under an organization policy, only the organization's allow rules grant access: local and kit allow rules are inactive, while deny rules still apply. `sbx policy ls` hides inactive rules unless you ask for them:

```bash
sbx policy check network --sandbox command-code api.commandcode.ai
sbx policy ls --include-inactive
# ... (1) corporate policy takes precedence over local allow rules.
```

Signing in with a personal account fixed it. If your organization governs sandboxes, the kit's allow list is not the final word, and the person who approves the egress may not be you.

---

## Updating Command Code

A new Command Code release does not need a new Dockerfile. The version is a kit argument, so you rebuild the same recipe with a different value. Two details shape how:

- **A sandbox never changes its image.** It keeps the image it was created with, so updating means removing it and creating it again. Your project lives on the host and isn't touched. Anything stored only inside the sandbox is lost, including Command Code's session history in `~/.commandcode`.
- **`latest` is cached.** The build resolves `latest` once and reuses the result while the kit files and arguments are unchanged. Rebuilding with `latest` gives you the old release.

So pass the new version explicitly. Going from 1.89.2 to 1.89.3:

```bash
npm view command-code version        # 1.89.3
sbx rm command-code
sbx create --name command-code --kit-arg version=1.89.3 \
  ./command-code ~/projects/hello-cmd
```

A new argument value only invalidates the cache from the npm layer onwards. The apt and `gh` layers are reused, so the rebuild takes about as long as the `npm install`. If you'd rather stay on `latest`, `sbx kit builder rm` clears the kit build cache, but it clears it for every kit and the next build starts from scratch.

---

## Choosing the base image

Docker suggests `docker.io/sbx/shell-image` as a starting point for custom agents. It already has the `agent` user, Node 22, ripgrep, `gh` and the shell setup, so this Dockerfile would shrink to an `npm i -g` and an entrypoint. I didn't use it. It is 2 GB, it ships a Docker CLI, Python and passwordless sudo that this agent doesn't need, and its `gh` is whatever version was current when Docker built it. The `node:22-bookworm-slim` build is about a quarter of that size, and every tool in it is one I chose.

Docker Hardened Images were the other candidate. I built the same kit on `dhi.io/node:22-debian12-dev`. It came out 24 MB larger, and Docker Scout reported 123 findings against 130. The 191 npm packages were identical, and so were the shared Debian package versions. The runtime DHI variant has no shell, so it fails the platform floor. The `-dev` variant keeps apt, bash and git, which is most of what hardening is meant to remove. For this kit the risk is in the npm dependency tree, not the base, so DHI isn't worth it here. It makes sense when you can use the runtime variant, or when you specifically need signed images, SBOMs and provenance.

---

## Publishing

A local directory is a valid kit reference, which is all you need while iterating. To share the kit, use the descriptor as the build file:

```bash
docker buildx build ./command-code \
  --file ./command-code/command-code.yaml \
  --platform linux/arm64,linux/amd64 \
  --tag docker.io/<NAMESPACE>/command-code:1.0.0 \
  --push

sbx run --name command-code-hub docker.io/<NAMESPACE>/command-code:1.0.0 ~/projects/hello-cmd
```

Pin the tag. Approvals are checked against the published descriptor, so if a later version adds a domain or a credential, `sbx` asks again before applying it. Command Code ships as an `UNLICENSED` npm package, so read its terms before pushing an image that contains it to a public registry. I haven't pushed one.

---

## Takeaways

- **A workload is two small files.** Most of my iterations went into learning the descriptor grammar, not fighting the tooling.
- **Each tool gets its own credential and its own hosts.** No token exists inside the VM, and the allow list shows exactly what each tool talks to.
- **`--yolo` belongs inside a boundary.** Inside a microVM, with a written allow list and a placeholder credential, permission prompts mostly get in the way.
- **Updates are rebuilds, not edits.** Recreate the sandbox with `--kit-arg version=x.y.z`; the Dockerfile stays the same.
- **Spend effort on the dependency tree, not the base.** A hardened base barely moved the numbers.

Some caveats. This kit is mine, not Docker's: the service name and network rules are my choices, and nobody has audited them. The kit format went from v1 to v3 in under a year, and the `kit` subcommands are still experimental. And organization governance can override your allow list without telling you.

Even so, Command Code now runs in a microVM on my machine. It talks to one domain, can't reach any host credentials, and doesn't stop to ask permission. That's a better default than running it directly.

---

*Tested with Docker Sandboxes v0.54.0 and Command Code v1.89.3 on macOS with Apple Silicon.*

---

## References

- [docker-sbx-command-code-kit on GitHub](https://github.com/matteobisi/docker-sbx-command-code-kit)
- [Docker Sandboxes documentation](https://docs.docker.com/ai/sandboxes)
- [sbx CLI reference](https://docs.docker.com/reference/cli/sbx/)
- [Kits](https://docs.docker.com/ai/sandboxes/customize/)
- [Build an agent workload](https://docs.docker.com/ai/sandboxes/customize/author/build-an-agent/)
- [Base images](https://docs.docker.com/ai/sandboxes/customize/author/base-images/)
- [Manage credentials](https://docs.docker.com/ai/sandboxes/configuration/credentials/)
- [Docker Sandbox Kit Specification v3](https://github.com/docker/sandbox-kit-spec/blob/main/docs/spec/SPEC-v3.md)
- [Command Code documentation](https://commandcode.ai/docs)
- [Docker Hardened Images](https://docs.docker.com/dhi/)
