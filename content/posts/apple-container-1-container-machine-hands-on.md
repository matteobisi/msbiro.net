---
title: "Apple container 1.0 and container machine: hands-on security test"
date: 2026-06-12T09:45:00+01:00
tags: [
  "apple", "apple-container", "container-machine", "macos", "macos26",
  "virtualization", "container-security", "developer-tools", "linux",
  "systemd", "cloud-native", "devsecops"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Hands-on Apple container 1.0 test of container machine on macOS, covering home-mount security, networking, systemd, and Docker or Podman alternatives."
canonicalURL: "https://www.msbiro.net/posts/apple-container-1-container-machine-hands-on/"
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
    alt: "Apple container 1.0 container machine security test on macOS"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

A few days ago during WWDC26, Apple released [`container` 1.0](https://github.com/apple/container/releases/tag/1.0.0). The release notes are short, but the important part is clear: Apple wants people to try the new [`container machine`](https://github.com/apple/container/blob/main/docs/container-machine.md) functionality.

As a team leader, when new products or tools enter the areas I work on, I like to spend some free time testing them directly. It helps me understand where they can be useful, where the limits are, and what security implications they may have for my engineering team or for customers.

I already tested early versions of Apple container when it was announced. This time I wanted to test the part that changes the developer workflow the most. Not only "can it run a container?", but "can it become a usable Linux environment on macOS, and what security boundary does it actually provide?"

If you prefer to see a breef explaination video about this, check this video from Apple Developers YouTube channel, otherwise go ahead and read what I fonund on my MacBook.

{{< youtube Q2xD6zkDz-s >}}

## The setup

The machine used for the test was running macOS 26.5.1. Apple container 1.0 was already installed.

```bash
container --version
# container CLI version 1.0.0 (build: release, commit: ee848e3)

container system status
# status             running
# apiserver.version  container-apiserver version 1.0.0
```

One visible change in 1.0 is the new configuration model. The service now reads TOML configuration instead of the older UserProperty backed settings. On my host, the merged system configuration included a dedicated `machine` section:

```toml
[machine]
cpus = 5
homeMount = "rw"
memory = "16gb"
```

That matters because these values became the defaults for the first machine I created, including the home mount mode.

## Creating the first machine

The quickstart is exactly what the documentation suggests:

```bash
container machine create alpine:latest --name dev
```

The first creation took about 38 seconds, including the image fetch. After creation:

```bash
container machine ls
```

```text
NAME  CREATED              IP            CPUS  MEMORY  DISK  STATE    DEFAULT
dev   2026-06-11 15:00:41  192.168.64.2  5     16G     75M   running  *
```

The machine was automatically selected as the default. Running commands inside it felt close to `docker exec`, but the target is a persistent Linux environment:

```bash
container machine run -n dev whoami
# matteo.bisi

container machine run -n dev pwd
# /Users/matteo.bisi/GitRepo/Personal

container machine run -n dev uname -a
# Linux dev 6.12.28 #1 SMP Tue May 20 15:19:05 UTC 2025 aarch64 Linux

container machine run -n dev -- cat /etc/os-release
# NAME="Alpine Linux"
# VERSION_ID=3.24.0
```

The first important detail is user mapping. Inside the machine, I was not `root`; I was my macOS user:

```bash
container machine run -n dev id
# uid=501(matteo.bisi) gid=20(dialout) groups=20(dialout)
```

The current working directory was also preserved because my home directory was mounted inside the machine. This is convenient, and it is also the most important security point in the whole feature.

## What is running inside

From the guest:

```bash
container machine run -n dev nproc
# 5

container machine run -n dev -- free -h
# Mem: 15.8G
# Swap: 0B
```

The machine had the 5 CPUs and 16 GB of memory configured in the host settings. PID 1 was `init`:

```bash
container machine run -n dev -- ps -o pid,user,comm | head
```

```text
PID   USER     COMMAND
1     root     init
7     root     getty
8     root     getty
```

The root filesystem is separate from macOS. The host home directory is mounted through `virtiofs`:

```bash
container machine run -n dev -- mount | grep /Users/matteo.bisi
# virtiofs on /Users/matteo.bisi type virtiofs (rw,relatime)
```

That is the core design. Apple container gives you a Linux VM like environment, then wires your macOS home directory into it so your repositories and dotfiles are immediately available.

## Lifecycle and resizing

The lifecycle commands work as expected:

```bash
container machine inspect dev
container machine logs dev
container machine stop dev
container machine run -n dev pwd
```

`run` boots the machine again if it is stopped. Configuration changes are persisted but apply after restart:

```bash
container machine set -n dev home-mount=ro cpus=2 memory=2G
container machine stop dev
container machine run -n dev nproc
# 2

container machine run -n dev -- free -h
# Mem: 2.1G
```

This part is clean. `container machine set` does not pretend to hot patch the running VM. It updates the machine configuration and tells you clearly that a stop and restart is required.

One note about `alpine:latest`: the machine worked for shell commands, but the logs repeatedly showed:

```text
can't run '/sbin/openrc': No such file or directory
```

So I would treat plain Alpine as a good quick shell test, not as the best service machine image. For long running services, use an image prepared with a real init system.

## The filesystem security test

The default `home-mount=rw` mode gives the machine full read and write access to the host home directory.

I verified this directly:

```bash
container machine run -n dev -- ls /Users/matteo.bisi/.ssh | head
# agent
# config
# config.backup
# id_ed25519
# id_ed25519.pub

container machine run -n dev -- head -1 /Users/matteo.bisi/.zshrc
# readable
```

I also wrote a file from inside the machine and saw it immediately on macOS:

```bash
container machine run -n dev /Users/matteo.bisi/.machine-home-probe.sh
# write_home:
# write-ok
```

This is not a bug. It is the documented convenience model. But it changes how I would use the feature. A trusted developer environment can use `rw`; an untrusted workload should not.

Then I tested `home-mount=ro`:

```bash
container machine set -n dev home-mount=ro
container machine stop dev
container machine run -n dev /Users/matteo.bisi/.machine-home-probe.sh
```

Observed:

```text
virtiofs on /Users/matteo.bisi type virtiofs (ro,relatime)
read_zshrc:
# readable
ssh_dir:
agent
config
config.backup
id_ed25519
id_ed25519.pub
write_home:
Read-only file system
write-failed
```

`ro` blocks writes, but it does not protect secrets from reads. Your SSH keys and dotfiles are still visible.

Then I tested `home-mount=none`:

```bash
container machine set -n dev home-mount=none
container machine stop dev
container machine run -n dev -- ls -la /Users/matteo.bisi
container machine run -n dev -- pwd
```

Observed:

```text
/Users/matteo.bisi exists as an empty root owned directory
/home/matteo.bisi
```

That is the mode I would use for anything I do not fully trust. It removes the direct host home exposure and gives the user a normal Linux home inside the machine.

Root inside the machine is also worth testing:

```bash
container machine run -n dev --root id
# uid=0(root) gid=0(root) groups=0(root)

container machine run -n dev --root touch /Users/matteo.bisi/.machine-root-test
ls -la ~/.machine-root-test
# -rw-r--r-- 1 matteo.bisi staff 0 ... /Users/matteo.bisi/.machine-root-test
```

Root in the guest did not create a root owned file on the macOS home mount. The file was mapped back to my macOS user. That is good, but it does not remove the bigger issue: with `rw`, a root process in the guest can still modify files your macOS user can modify.

## The network security test

The machine gets an address on a private `192.168.64.0/24` network:

```text
eth0: 192.168.64.x/24
default gateway: 192.168.64.1
DNS: 192.168.64.1
domain/search: machine
```

Outbound HTTPS worked:

```bash
container machine run -n dev wget -q -O- -T 5 https://ifconfig.me
```

The guest could also reach a service I exposed on the host gateway:

```bash
# On macOS
python3 -m http.server 8099 --bind 0.0.0.0

# From the machine
container machine run -n dev -- nc -z -w 3 192.168.64.1 8099
# reachable
```

The opposite direction also worked. I started a listener inside the machine, then connected from macOS:

```bash
container machine run -n dev -d /Users/matteo.bisi/.machine-nc-listener.sh
nc -w 3 192.168.64.6 8080
# hello-from-container-machine
```

This is important. `container machine` gives you a VM boundary and a separate Linux network stack, but it is not a locked down network sandbox by default. The guest has outbound access, the guest can reach host services exposed on the gateway, and the host can reach guest services on the machine IP.

That is fine for a developer machine. It is not the same security model as a policy controlled agent sandbox.

## Running a real systemd machine

After the Alpine test, I wanted to verify the part that makes `container machine` different from a normal command runner. The documentation says any Linux image with `/sbin/init` can become a machine, so I tested that with Ubuntu 24.04 and systemd.

The image was built locally with `container build`:

```dockerfile
FROM ubuntu:24.04

ENV container=container
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates curl dbus iproute2 iputils-ping net-tools \
      openssh-server procps sudo systemd systemd-sysv vim-tiny wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN : > /etc/machine-id && \
    : > /var/lib/dbus/machine-id

RUN systemctl set-default multi-user.target
```

Then:

```bash
container build --progress plain -t local/ubuntu-machine:1.0-test ./ubuntu-machine

container machine create local/ubuntu-machine:1.0-test \
  --name ubuntu-systemd \
  --cpus 2 \
  --memory 4G \
  --home-mount=ro
```

The machine started correctly:

```text
NAME            IP            CPUS  MEMORY  DISK  STATE
ubuntu-systemd  192.168.64.8  2     4G      238M  running
```

Inside:

```bash
container machine run -n ubuntu-systemd -- cat /etc/os-release | head -3
# PRETTY_NAME="Ubuntu 24.04.4 LTS"

container machine run -n ubuntu-systemd -- ps -p 1 -o pid,comm,args
# PID  COMMAND  COMMAND
# 1    systemd  /sbin/init

container machine run -n ubuntu-systemd systemctl is-system-running
# running
```

Then I started SSH as a real systemd service:

```bash
container machine run -n ubuntu-systemd --root systemctl start ssh
container machine run -n ubuntu-systemd systemctl is-active ssh
# active

nc -z -w 3 192.168.64.8 22
# Connection to 192.168.64.8 port 22 [tcp/ssh] succeeded
```

That is the practical value of container machines. You can keep a Linux environment around, run real services under systemd, and still use your macOS editor against the same repository tree if you choose to mount it.

## A small CLI rough edge

I observed one behavior that I would retest before relying on complex command strings:

```bash
container machine run -n dev -- sh -c 'echo hi'
```

Expected:

```text
hi
```

Observed:

```text

```

Another probe compressed repeated spaces:

```bash
container machine run -n dev -- echo 'a b   c'
# a b c
```

This looks like argv forwarding may not preserve boundaries exactly in every case. The workaround is simple: put complex commands in a script available inside the machine, then execute the script. That is what I used for the filesystem and network probes.

## How I would use it

For a trusted developer environment, `container machine` is genuinely useful. I can see myself keeping separate machines for different targets, for example Ubuntu with systemd, Alpine for lightweight checks, or Debian for package compatibility.

For security sensitive use, the default is too permissive for my taste. The home mount decides the risk level:

| Mode | What happens | Security interpretation |
|------|--------------|-------------------------|
| `rw` | Full host home is readable and writable | Best convenience, biggest blast radius |
| `ro` | Full host home is readable, writes blocked | Useful for trusted reads, still exposes secrets |
| `none` | Host home is not mounted | Best default for untrusted workloads |

Network behavior follows the same pattern. It is practical, but not restrictive. If a service listens inside the machine, macOS can reach it by the machine IP. If a service listens on the host gateway, the machine can reach it. Outbound internet worked in my test.

## Final thoughts

Apple container 1.0 feels much more interesting with `container machine` than as only another way to run OCI containers on macOS. I appreciate that Apple is making this kind of tool available directly on macOS, with a native CLI and a workflow that is simple enough to understand after a few commands. The usability is good if you are comfortable in a shell.

The feature also solves a practical problem. Sometimes I need to run a Linux tool, test a package, or keep a small Linux environment available without starting a full traditional VM every time. For that use case, `container machine` is useful. It gives me a persistent Linux environment, fast startup, direct access from the terminal, and enough flexibility to run a real systemd based machine when needed.

The security story is honest if you understand the defaults. The VM boundary is real, PID 1 and systemd can work, root inside the guest does not become root on the host home mount, and resource configuration behaves cleanly across restarts.

The default home mount is the part to treat carefully. `rw` is great for trusted personal development. It is not what I would use for random scripts, untrusted code, or autonomous agents. For that, start with `home-mount=none`, add only the minimum files you need, and expose services deliberately.

At the same time, there are more established alternatives. Docker Desktop is a much more complete product, even if licensing matters depending on your context. Docker Sandboxes is on another level if the goal is running AI agents with a stronger feature set around usability and isolation, and I already covered that in my [hands on test of Docker Sandboxes](https://www.msbiro.net/posts/docker-sandboxes-ai-agents/). Podman Desktop is also a mature option if you want a desktop experience and a broader container workflow.

So my short version is this: Apple container 1.0 and `container machine` are useful, viable, and technically interesting. They are also basic tools that assume the user is comfortable with the shell. If you want a native macOS CLI to run containers or Linux environments without managing full VMs, it is worth trying. If you need a GUI, richer product features, stronger sandbox ergonomics, or a polished day to day container platform, you should probably look at Docker Desktop, Docker Sandboxes, Podman Desktop, or another more complete solution.
