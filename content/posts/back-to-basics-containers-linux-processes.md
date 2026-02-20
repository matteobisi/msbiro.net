---
title: "Back to Basics: Why Containers Are Just Fancy Linux Processes"
date: 2026-02-20T06:31:29Z
tags: [
  "cloud-native", "kubernetes", "cybersecurity",
  "open-source", "devops", "containers", "linux",
  "processes", "permissions", "security-context",
  "namespaces", "cgroups"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Containers are Linux processes with namespaces and cgroups, nothing more. This article breaks down what Kubernetes securityContext, resource limits, and container escapes actually do at the kernel level, and shows you how to debug containers using standard Unix tools like nsenter and /proc."
canonicalURL: "https://www.msbiro.net/posts/back-to-basics-containers-linux-processes/"
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
    alt: "Linux namespaces and cgroups visualized as process isolation layers"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

The path into platform engineering has changed. Many engineers today start their careers working directly with Kubernetes, writing YAML and managing Helm charts before they ever spend extended time at a Linux terminal. The tooling is so well-abstracted that you can be genuinely productive for months before the underlying system ever becomes relevant. That is a real achievement for the ecosystem.

The gap shows up at the worst moments, though: a container crashes with a permission error, a security team flags a pod running as root, a privilege escalation CVE lands and it is not clear whether the cluster is exposed. These are Linux problems, and they are much easier to reason about once you understand what the YAML actually maps to at the kernel level. I have been in those conversations many times, and I always come back to the same set of fundamentals.

This article is the second in my "Back to Basics" series. In December, I wrote about [SSH hardening configuration](https://www.msbiro.net/posts/back-to-basics-sshd-hardening/). This piece continues that journey, away from the YAML and back to the Linux fundamentals that make everything else possible.

**Containers are just Linux processes with extra isolation.** Everything Kubernetes does with permissions, users, and security contexts is built on top of concepts that have existed in Unix systems for decades. Understanding these basics makes container security much less mysterious, and debugging much faster.

---

## The Unix Foundation

This section covers the three fundamental concepts that underpin all Linux security: file permissions, process identity, and special permission bits.

### File Permissions

Unix permissions from 1971 still power everything today. Every file has permissions for three categories: **Owner (u)**, **Group (g)**, and **Others (o)**. Each has three possible permissions: **Read (r)**, **Write (w)**, **Execute (x)**.

```bash
$ ls -l /etc/passwd
-rw-r--r-- 1 root root 2875 Jan 15 09:23 /etc/passwd
```

Breaking down `-rw-r--r--`:
- First character: `-` means regular file
- `rw-`: Owner (root) can read and write
- `r--`: Group (root) can only read  
- `r--`: Others can only read

Every container image, volume mount, and application file uses these nine permission bits.

Use `chmod` to modify permissions:

```bash
# Symbolic
chmod u+x script.sh        # Add execute for owner
chmod o-w sensitive.conf   # Remove write for others

# Numeric (4=read, 2=write, 1=execute)
chmod 644 file.txt         # rw-r--r--
```

`chmod 777` means `rwxrwxrwx`: full permissions for everyone. **Avoid this in production.**

### Process Identity

Every running process has an identity that determines what it can do. When you run a command, it uses your user ID (UID):

```bash
$ id
uid=1000(matteo) gid=1000(matteo) groups=1000(matteo),27(sudo)
```

Processes have two UIDs:
- **Real UID**: Who you actually are
- **Effective UID**: What permissions you have right now

Usually they match. When they differ (via setuid), you can gain elevated privileges.

The `/proc` filesystem exposes every process:

```bash
# Current shell PID
$$

# View process UIDs
cat /proc/$$/status | grep -E 'Uid|Gid'
# Uid:    1000    1000    1000    1000
# Gid:    1000    1000    1000    1000
```

Four values shown: Real UID, Effective UID, Saved UID, Filesystem UID.

### Special Permission Bits

Three special bits have major security implications.

**Setuid (Set User ID)** makes an executable run as the file's owner, not the executing user:

```bash
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 Jan 25  2023 /usr/bin/passwd
```

The `s` in the owner execute position is setuid. When you run `passwd`, it runs as root (UID 0), allowing it to modify `/etc/shadow`. Setgid works the same for groups.

**Sticky Bit** on directories means only owners can delete their own files:

```bash
$ ls -ld /tmp
drwxrwxrwt 18 root root 4096 Feb 18 10:00 /tmp
```

The `t` prevents users from deleting others' files in `/tmp`.

**Security Impact**: Setuid binaries are a major attack surface. A vulnerability in a setuid binary lets attackers escalate from regular user to root. Hardened systems minimize or eliminate them.

---

## From Processes to Containers

Now that we understand Unix permissions and process identity, let us see how containers use these features and how Kubernetes `securityContext` maps directly to them.

### What Is a Container?

A container is a process (or process group) with Linux kernel isolation features:

1. **Namespaces**: Isolate what processes see (PID, network, filesystem, hostname, IPC, UIDs)
2. **Cgroups**: Limit resource usage (CPU, memory, I/O)
3. **Capabilities**: Fine-grained privileges instead of all-or-nothing root
4. **Seccomp**: Filter system calls
5. **AppArmor/SELinux**: Mandatory access controls

It is still just a process. The kernel does not have a concept of "container"; only namespaces and cgroups applied to processes.

### Prove It: See the Container as a Process

The fastest way to understand this is to observe it directly. Start a container and immediately find it on the host:

```bash
# Start a container in the background
docker run -d --name demo nginx

# Find it on the host process table — it's just a process
ps aux | grep nginx
# root  18423  nginx: master process
# nginx 18424  nginx: worker process

# Get the exact PID from Docker
docker inspect demo --format '{{.State.Pid}}'
# 18423

# Inspect its identity and capabilities directly from /proc
cat /proc/18423/status | grep -E 'Uid|Gid|Cap'
# Uid:  0  0  0  0       <- root inside and outside (dangerous)
# CapEff: 00000000a80425fb  <- capabilities bitmask

# See its namespaces — each is a different file descriptor
ls -la /proc/18423/ns/
# lrwxrwxrwx ... net -> net:[4026532345]
# lrwxrwxrwx ... pid -> pid:[4026532347]
# lrwxrwxrwx ... mnt -> mnt:[4026532346]
```

That nginx running as UID 0 inside the container is UID 0 on the host. No magic, no VM boundary; the same kernel, the same user table.

### Namespaces: What Each One Isolates

Linux has eight namespace types. Each one limits what a process can see, not what the host can see:

| Namespace | Isolates | Kubernetes Example |
|-----------|----------|--------------------|
| `pid` | Process tree | Pod sees PID 1; host sees PID 18423 |
| `net` | Network stack, interfaces, ports | Each pod gets its own `eth0` |
| `mnt` | Filesystem mounts | Container sees its own rootfs |
| `uts` | Hostname and domain name | `hostname` returns the pod name |
| `ipc` | IPC, shared memory segments | Pods cannot share memory by default |
| `cgroup` | Cgroup hierarchy view | Pod sees only its own resource limits |
| `user` | UID/GID mappings | Rootless containers (Kubernetes 1.33+) |
| `time` | System time offsets (CLOCK_MONOTONIC / CLOCK_BOOTTIME) | Added in Linux 5.6 |

You can create namespaces manually with `unshare`; there is no container runtime needed:

```bash
# Create a new PID namespace: this bash process becomes PID 1
sudo unshare --pid --fork --mount-proc bash
echo $$
# 1   <- you are "PID 1" inside this namespace

# The host still sees it as its real PID
# Open another terminal: ps aux | grep bash -> real PID, e.g. 19001
```

This is what a container runtime does: wrap a process in multiple namespaces simultaneously.

### Cgroups: Resource Limits Are Kernel Enforced

When you write this in a Kubernetes manifest:

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
```

Kubernetes translates it into cgroup entries that the kernel enforces directly:

```bash
# Find the container's cgroup path
cat /proc/18423/cgroup
# 0::/kubepods/burstable/pod<uid>/<container-id>

# The kernel enforces the memory limit here
cat /sys/fs/cgroup/kubepods/burstable/pod<uid>/<container-id>/memory.max
# 536870912   <- 512Mi in bytes

# CPU quota (500m = 50% of one core = 50000 microseconds per 100ms period)
cat /sys/fs/cgroup/kubepods/burstable/pod<uid>/<container-id>/cpu.max
# 50000 100000
```

When Kubernetes OOMKills a pod, it is not Kubernetes that kills it; it is the kernel's cgroup memory controller sending `SIGKILL` when the process exceeds `memory.max`. Understanding this makes OOMKill events much easier to diagnose.

### Kubernetes securityContext Decoded

Every Kubernetes `securityContext` field maps to a Linux feature.

**runAsUser and runAsGroup: Setting Process Identity**

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
```

Equivalent to:
```bash
su -c "myapp" -s /bin/sh user1000
# The process gets UID 1000 and primary GID 3000
```

Or in Dockerfile:
```dockerfile
USER 1000:3000
```

Regular users start at UID 1000. Using non-zero UIDs prevents accidental host user mapping. Setting `runAsGroup` explicitly controls the primary GID, which affects file creation permissions inside the container.

**runAsNonRoot: Prevent Root Execution**

```yaml
securityContext:
  runAsNonRoot: true
```

Kubernetes rejects pods trying to run as UID 0. This blocks the easiest container escape path: if you are root inside the container and escape, you become root on the host.

**allowPrivilegeEscalation: Block Setuid**

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

Prevents processes from gaining more privileges than they started with. Even with setuid binaries like `sudo`, the process cannot escalate. This uses the Linux `no_new_privs` flag (available since 2012).

**readOnlyRootFilesystem: Remount Read-Only**

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Equivalent to `mount -o remount,ro /`. The container can still write to volumes and `/tmp`, but base image files are immutable. This prevents attackers from installing malware.

**capabilities: Fine-Grained Privileges**

Traditional Unix is binary: root or not-root. Linux Capabilities split root into smaller pieces:

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
```

This drops all capabilities, then adds only `CAP_NET_BIND_SERVICE` for binding ports below 1024. Instead of running as root just to bind port 80, you only get the specific capability needed.

**fsGroup: Volume Ownership**

```yaml
securityContext:
  fsGroup: 2000
```

Kubernetes runs `chown -R :2000` (group-only) on the volume mount point. This lets the container user (UID 1000) access files via group ownership (GID 2000).

**Limitation**: Only works on volumes supporting ownership changes (emptyDir, configMap, secret). HostPath and NFS may ignore this.

### Container Escape Risks

The Unix concepts from the first section are exactly what attackers exploit. A container running as root with excessive capabilities is not isolated; it is a root shell with extra steps.

**Privileged Mode**

```yaml
securityContext:
  privileged: true
```

This gives the container:
- Full host device access (`/dev/*`)
- Kernel parameter modification
- No seccomp filtering
- All capabilities

**From inside a privileged container, you can access the host:**

```bash
# Inside container
mkdir /mnt/host
mount /dev/sda1 /mnt/host
chroot /mnt/host
# Now you are root on the host
```

Never use `privileged: true` in production. It breaks all isolation.

Real-world examples of what happens when root-in-container meets kernel bugs or misconfigurations: **CVE-2019-5736** (runc container escape via `/proc/self/exe`) and **CVE-2022-0492** (cgroups v1 escape via `notify_on_release`) both allowed full host root access. Both would have been mitigated by `runAsNonRoot: true` combined with `allowPrivilegeEscalation: false`; the container would have been running as an unprivileged UID, making the privilege escalation path either impossible or harmless. The pattern is not new. I covered three more recent runc CVEs with the same escape potential (all disclosed in November 2025) in [this article](https://www.msbiro.net/posts/runc-container-breakout-vulnerabilities-2025/).

**HostPath Mounts**

```yaml
volumes:
  - name: host-root
    hostPath:
      path: /
```

Mounting the host filesystem gives direct host access. If the container runs as UID 0, it can modify `/etc/shadow`, SSH keys, or systemd services.

**Defense**: Always use `readOnly: true` for HostPath, and never run as root.

**Setuid Binaries in Images**

Standard images contain many setuid binaries:

```bash
$ docker run --rm ubuntu:latest find / -perm -4000 -type f 2>/dev/null
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/mount
... (many more)
```

Each is a privilege escalation vector.

**Defense**: Use distroless images without setuid binaries:

```dockerfile
FROM gcr.io/distroless/static-debian12:nonroot
```

**Running as Root**

Running containers as root is one of the most common configurations seen in early Kubernetes deployments. When a container running as UID 0 escapes (via kernel bugs, misconfigured volumes, or privileged mode), it becomes root on the host. Setting `runAsUser: 1000` is a straightforward step that removes that risk entirely.

### Debugging Containers Using Unix Tools

All the tools you need already exist on the host. No special container tooling required.

**Inside Containers**

Standard Unix tools work:

```bash
id            # Check current user
ps aux        # List processes
ls -la /app/  # Check permissions
lsof          # Open files
ss -tlnp      # Network connections
```

**On the Host**

```bash
# List containers
crictl ps

# Get container PID
crictl inspect <container-id> | grep pid

# Inspect namespaces
ls -la /proc/<pid>/ns/

# View process tree
pstree -p <pid>

# Check capabilities
cat /proc/<pid>/status | grep Cap
```

**Enter a Container's Namespaces Without kubectl exec**

When `kubectl exec` is unavailable (no shell in the image, exec disabled by policy, or the container has crashed), `nsenter` lets you enter any combination of the container's namespaces from the host:

```bash
# Full shell inside all container namespaces
nsenter -t <pid> --mount --uts --ipc --net --pid -- bash

# Enter only the network namespace — useful for tcpdump or ss
nsenter -t <pid> --net -- ss -tlnp

# Run a command in the container's mount namespace to inspect its filesystem
nsenter -t <pid> --mount -- ls -la /app/
```

`nsenter` is particularly useful on nodes where `kubectl exec` is rate-limited, on distroless containers with no shell, or when debugging what a container actually sees vs. what the host sees.

**Decoding Capabilities**

Capability values in `/proc/<pid>/status` are hex bitmaps:

```bash
# Decode capabilities
capsh --decode=0000003fffffffff

# Or use getpcaps
getpcaps <pid>
```

### Rootless Containers and User Namespaces

Rootless containers run as unprivileged host users.

Podman supports this natively:

```bash
# As regular user, no sudo needed
podman run nginx
```

The container has UID 0 inside its namespace, but maps to your regular UID on the host. If the container escapes, it has no host privileges.

In Kubernetes 1.33+, user namespace support is enabled by default (no feature gate needed) but pods must still opt in with `spec.hostUsers: false`. I previously covered how [user namespaces remap container root to unprivileged host users](/posts/kubernetes-133-user-namespace-isolation-security-matters/). Instead of preventing escapes, user namespaces make them harmless.

---

## Conclusion

Containers are not magic or virtual machines. They are **Linux processes with isolation**.

Kubernetes security settings map directly to Unix features:

| Kubernetes | Linux Equivalent |
|------------|------------------|
| `runAsUser: 1000` | Process UID |
| `runAsGroup: 3000` | Process primary GID |
| `allowPrivilegeEscalation: false` | `no_new_privs` flag (blocks setuid) |
| `readOnlyRootFilesystem: true` | Read-only mount |
| `capabilities` | Fine-grained privileges |
| `seccompProfile` | Syscall filter (seccomp BPF) |
| `privileged: true` | Broken isolation |

When YAML abstractions fail, debug at the Linux level. Understanding these foundations helps you understand attack vectors and build secure systems.

Master the basics: keep containers unprivileged, minimize setuid binaries, remember it is all just processes, users, and permissions.

In the next article in this series, I plan to go deeper on Linux cgroups: how resource limits are enforced at the kernel level, what actually happens during a Kubernetes OOMKill, and how to read cgroup v2 hierarchies to diagnose resource contention.

---

## Further Reading

- [Linux Capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Namespaces in Operation](https://lwn.net/Articles/531114/)
- [Kubernetes Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [Podman Rootless](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
