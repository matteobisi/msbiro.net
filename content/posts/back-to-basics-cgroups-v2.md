---
title: "Back to Basics: cgroups v2 and What Actually Happens When Kubernetes Kills Your Pod"
date: 2026-09-14T08:00:00+01:00
tags: [
  "cloud-native", "kubernetes", "cybersecurity",
  "open-source", "devops", "containers", "linux",
  "cgroups", "oomkill", "resource-limits", "qos",
  "memory-management", "cpu-throttling"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "cgroups v2 explained from the ground up: how Kubernetes resource limits become kernel-enforced numbers, what actually happens step by step during an OOMKill, and how to read the cgroup v2 hierarchy to diagnose memory and CPU contention instead of guessing."
canonicalURL: "https://www.msbiro.net/posts/back-to-basics-cgroups-v2/"
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
    alt: "cgroup v2 hierarchy visualized as a tree with memory and cpu controller limits"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Every platform engineer has seen it. A pod is running fine for weeks, then one day it restarts and `kubectl describe` reports `OOMKilled`. Someone says "add more memory." The limit goes up, the pod stops dying, and nobody asks what actually happened under the hood.

That is the gap this series exists to close. In the [first article](https://www.msbiro.net/posts/back-to-basics-sshd-hardening/) I hardened an SSH daemon and explained why the defaults are insecure. In the [second](https://www.msbiro.net/posts/back-to-basics-containers-linux-processes/) I showed that containers are ordinary Linux processes wrapped in namespaces and cgroups. In the [third](https://www.msbiro.net/posts/back-to-basics-tls-pki/) I stripped TLS down to its raw structures. This fourth article goes deeper into the cgroups layer I only touched briefly before, because it is where resource limits actually live, and where most Kubernetes "mystery" incidents are decided.

Open a terminal on any Linux host running Kubernetes, follow along, and by the end you will be able to read the exact numbers your `resources.limits` YAML became, watch the kernel enforce them, and explain an OOMKill from first principles instead of guessing.

---

## The Limit You Wrote, Not the Limit You See

When you write this in a pod manifest:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

You are not asking Kubernetes to do anything directly. You are giving Kubernetes a set of numbers that it hands to the kernel, and the kernel is the entity that actually enforces them. Kubernetes does not kill your pod when it runs out of memory. The Linux kernel's memory controller does. Understanding that distinction is the entire difference between guessing and diagnosing.

The mechanism is cgroups, control groups, a kernel feature that organizes processes into a tree and applies resource controls to each node of that tree. Every pod, and every container inside a pod, is a node in that tree.

---

## cgroups v1 vs v2: Why the Hierarchy Matters

For years Linux had cgroups v1, which is really several independent hierarchies. Memory was controlled in one place, CPU in another, I/O in a third, each mounted separately under `/sys/fs/cgroup`. This worked but made the relationships between controllers unclear, and it forced tools like the container runtimes and systemd to coordinate carefully to avoid two managers fighting over the same resources.

cgroups v2 replaces that with a single unified hierarchy. There is one tree under `/sys/fs/cgroup`, and every controller (memory, cpu, io, pids, and so on) is managed within that one tree. Controllers are enabled per subtree by writing to a control file, which gives you a clear, readable model: a process belongs to exactly one cgroup, and that cgroup's position in the tree determines which limits apply to it.

This matters practically because the Linux kernel has been migrating to v2 for years, and Kubernetes made cgroup v2 the standard from version 1.25 onward. When you debug a node today, you are almost always debugging a v2 hierarchy.

Check which version a host runs:

```bash
# v1 mounts several subdirectories; v2 mounts a single hierarchy
mount | grep cgroup

# A v2 system shows one unified mount and a controllers file
cat /sys/fs/cgroup/cgroup.controllers
# cpuset cpu io memory hugetlb pids rdma misc
```

If you see `cpuset cpu io memory hugetlb pids` listed in `cgroup.controllers`, you are on v2.

---

## Reading the Hierarchy by Hand

The best way to understand cgroups is to look at them directly. Every cgroup is a directory, and the kernel exposes its configuration and live state as plain files.

Create a test cgroup and inspect it (as root):

```bash
# Create a cgroup directory
mkdir /sys/fs/cgroup/demo

# List its default files
ls /sys/fs/cgroup/demo
```

You will see files like `cgroup.procs`, `memory.max`, `cpu.max`, `pids.max`, and many more. Each one is a live interface to the kernel.

The most important files to understand first:

| File | What it controls |
| --- | --- |
| `cgroup.procs` | The PIDs that belong to this cgroup |
| `cgroup.controllers` | Controllers available at this level |
| `cgroup.subtree_control` | Controllers enabled for children of this cgroup |
| `memory.max` | The hard memory limit, in bytes |
| `memory.current` | Memory currently in use, in bytes |
| `memory.events` | Counters for events like `oom` and `oom_kill` |
| `cpu.max` | The CPU limit, as `quota period` in microseconds |
| `cpu.weight` | Relative CPU share when the host is contended |
| `pids.max` | Maximum number of processes |

Move the current shell into the cgroup and set a memory limit:

```bash
# Put this shell into the demo cgroup
echo $$ > /sys/fs/cgroup/demo/cgroup.procs

# Confirm the kernel now sees it there
cat /proc/self/cgroup
# 0::/demo

# Set a 64 MiB memory limit
echo $((64 * 1024 * 1024)) > /sys/fs/cgroup/demo/memory.max
cat /sys/fs/cgroup/demo/memory.max
# 67108864
```

From this point, the shell and everything it starts are subject to a hard 64 MiB ceiling enforced by the kernel. There is no userspace policy involved anymore.

---

## Memory Limits in Detail

Memory control in cgroup v2 is richer than a single ceiling. There is a spectrum of files, each with a different behaviour:

| File | Behaviour |
| --- | --- |
| `memory.max` | Hard limit. Exceeding it triggers reclaim, and if reclaim fails, the OOM killer. |
| `memory.high` | Soft limit. Exceeding it throttles and reclaims aggressively, but does not OOM kill. |
| `memory.low` | Best-effort protection. Memory above this is reclaimed first under pressure. |
| `memory.min` | Hard protection. Memory below this is never reclaimed. |

The distinction between `max` and `high` is the source of much confusion. A pod that exceeds `memory.high` slows down. A pod that exceeds `memory.max` dies. When Kubernetes sets `limits.memory`, it writes to `memory.max`, which is why a pod crossing its limit is killed rather than throttled.

Watch the events happen live. Set a low `memory.high` and allocate memory, and you will see the kernel start reclaiming long before anything is killed:

```bash
# Set a soft limit and a hard limit
echo $((32 * 1024 * 1024)) > /sys/fs/cgroup/demo/memory.high
echo $((64 * 1024 * 1024)) > /sys/fs/cgroup/demo/memory.max

# Allocate memory and observe the counters
cat /sys/fs/cgroup/demo/memory.current
cat /sys/fs/cgroup/demo/memory.events
# low 0 high 123 max 0 oom 0 oom_kill 0
```

The `memory.events` file is the first place to look after an incident. The `oom` counter increments when the kernel considers killing something, and `oom_kill` increments when it actually does.

---

## CPU Limits and Throttling

CPU control is different from memory. You cannot "use up" CPU the way you exhaust memory; instead the kernel meters it over time. A CPU limit of `500m` in Kubernetes means half of one CPU core, which the kernel represents as a quota over a period.

In cgroup v2, `cpu.max` uses two numbers: a quota and a period, both in microseconds. Kubernetes writes `500m` as `50000 100000`, meaning the cgroup may run for 50 milliseconds out of every 100 millisecond period:

```bash
cat /sys/fs/cgroup/demo/cpu.max
# 50000 100000   <- 50% of one core
```

When the processes in the cgroup use up their quota before the period ends, they are throttled. The cgroup does not die; it simply gets fewer cycles. This is why a CPU-throttled pod looks "slow" rather than "crashed," and it is much harder to spot than an OOMKill because nothing restarts.

The throttling is recorded in `cpu.stat`:

```bash
cat /sys/fs/cgroup/demo/cpu.stat
# nr_periods 1523
# nr_throttled 45
# throttled_usec 183904
```

`nr_throttled` above zero means the pod is hitting its CPU ceiling. If a service is mysteriously slow while its memory looks fine, this is the file to read.

`cpu.weight` is the other side of the coin. When `limits.cpu` is not set, Kubernetes uses `requests.cpu` to assign a weight, which only matters when the host is contended. A weight of 100 is the default; higher weights get a larger share of a busy CPU, but none of it is a hard ceiling.

---

## How Kubernetes Maps Your YAML to cgroups

Kubernetes does not invent its own resource model. It translates pod manifests into cgroup placements and file writes, and the mapping is deterministic.

On a node using the systemd cgroup driver, the hierarchy looks like this:

```
/sys/fs/cgroup/
├── kubepods.slice/
│   ├── kubepods-burstable.slice/
│   │   └── kubepods-burstable-pod<uid>.slice/
│   │       └── cri-containerd-<container-id>.scope/
│   ├── kubepods-besteffort.slice/
│   │   └── ... best-effort pods ...
│   └── kubepods-guaranteed.slice/  (or placed directly under kubepods.slice)
```

Each container gets its own cgroup, and the pod-level cgroup holds the containers together. This is why a pod's resource limits apply to the combined memory of all its containers.

Find the cgroup for a running container directly:

```bash
# Get the container's PID
crictl inspect <container-id> | grep -i pid

# See which cgroup the kernel assigned it to
cat /proc/<pid>/cgroup
# 0::/kubepods.slice/kubepods-burstable.slice/.../cri-containerd-<id>.scope

# Read the exact limit Kubernetes wrote
cat /sys/fs/cgroup/kubepods.slice/kubepods-burstable.slice/.../memory.max
```

The number you find there is your `limits.memory` value in bytes. It is not a coincidence; it is the literal write that Kubernetes performed on your behalf.

The placement is not random either. It reflects the pod's Quality of Service class:

| QoS class | How it is placed | OOM score adjustment |
| --- | --- | --- |
| Guaranteed | requests equal limits | `-997` (last to be killed) |
| Burstable | requests lower than limits | between `-997` and `1000` |
| BestEffort | no requests or limits | `1000` (first to be killed) |

BestEffort pods are the first victims of a node-wide memory crunch, and Guaranteed pods are protected as long as possible. The `oom_score_adj` value is what the kernel uses to choose the victim when the whole node runs out, a separate decision from the per-cgroup `memory.max` limit.

---

## What Actually Happens During an OOMKill

Now the sequence makes sense. When a container's memory use climbs past `memory.max`, the kernel does not kill anything immediately. It follows a progression:

1. The process allocates memory, and `memory.current` rises.
2. When `memory.current` reaches `memory.max`, the kernel attempts to reclaim memory: it drops clean page cache, compresses memory if enabled, and writes back dirty pages.
3. If reclaim succeeds, the process continues, possibly slower.
4. If reclaim cannot free enough memory, the kernel selects a process in the cgroup to kill and sends `SIGKILL`.
5. The `oom_kill` counter in `memory.events` increments, and the kernel logs the event to `dmesg`.

The process that gets killed is not always the one that caused the pressure; the kernel picks based on its own heuristics. What is guaranteed is that the kill happens inside the cgroup that hit its own `memory.max`, which is what keeps a misbehaving pod from taking down the whole node.

Kubernetes then notices the container's main process died, marks the pod as `OOMKilled`, and restarts it according to `restartPolicy`. This is why the pod "restarts" and why the restart count climbs while the workload itself keeps hitting the same ceiling.

The kernel-side evidence is all in one place:

```bash
# The kernel's own record of the kill
dmesg | grep -i oom

# The cgroup's event counters
cat /sys/fs/cgroup/<pod-cgroup>/memory.events
# oom 1
# oom_kill 1
```

If `oom_kill` is nonzero, the pod did not crash randomly. It hit a limit, and the kernel enforced it. The YAML was not a suggestion.

---

## Diagnosing Resource Contention Instead of Guessing

The whole point of reading cgroups directly is that you stop guessing. Every common "mystery" has a file that answers it.

**"Why was my pod killed?"** Read `memory.events`. If `oom_kill` is nonzero, it hit `memory.max`. Compare that against the pod's actual usage with `memory.current` over time, or with the node-level view.

**"Why is my pod slow but not dead?"** Read `cpu.stat`. A nonzero `nr_throttled` means CPU throttling. Either raise the limit or the request, or investigate why the workload is bursty.

**"Why did the whole node freeze?"** Look at the node's top-level `memory.current` and `memory.events` at the root cgroup, and check whether a BestEffort pod is consuming everything while Guaranteed pods stay protected.

A quick manual audit of the live hierarchy:

```bash
# Walk the tree and show memory usage and limits
find /sys/fs/cgroup/kubepods.slice -name memory.current -print \
  -exec sh -c 'printf "%s " "$1"; cat "$1"' _ {} \; | sort -k2 -n | tail -20
```

And the same idea for CPU throttling across the node:

```bash
find /sys/fs/cgroup/kubepods.slice -name cpu.stat -print \
  -exec grep -H throttled {} \;
```

These are the raw facts. Kubernetes dashboards and observability tools surface the same numbers with more polish, but when the dashboard disagrees with reality, the file is the source of truth.

---

## Conclusion

An OOMKill is not a mystery and not a Kubernetes decision. It is the Linux kernel's memory controller enforcing a limit that you wrote in a YAML file, translated into a number in a cgroup directory, and enforced on a live process tree. Once you can read `memory.max`, `memory.current`, `memory.events`, and `cpu.stat`, the incidents stop being guesses and become arithmetic.

The same shift happened in the earlier articles of this series: SSH stopped being a pile of defaults, containers stopped being black boxes, TLS stopped being a padlock icon. cgroups is the same lesson one layer down. The abstraction is comfortable, but the files are where the truth lives.

In the next article in this series, I plan to go back under the hood of Linux networking, how packets actually flow through iptables and nftables, and what Kubernetes Services and kube-proxy are really doing to your traffic.

---

## Further Reading

- [Kernel documentation: Control Group v2](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)
- [Kubernetes: Managing Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Kubernetes: Configure Quality of Service for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)
- [Kubernetes: Configure a kubelet image and cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/)
- [systemd: Control Group Interfaces](https://systemd.io/CGROUP_DELEGATION/)
- [Kernel documentation: Memory Resource Controller](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html#memory-interface-files)
