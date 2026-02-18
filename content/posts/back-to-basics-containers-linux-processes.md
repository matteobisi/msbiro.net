---
title: "Back to Basics: Why Containers Are Just Fancy Linux Processes"
date: 2026-02-18T17:31:29Z
tags: [
  "cloud-native", "kubernetes", "cybersecurity",
  "open-source", "devops", "containers", "linux", 
  "processes", "permissions", "security-context"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Demystifying containers by understanding Linux processes, user permissions, and security contexts. Learn what Kubernetes securityContext actually does under the hood and why container escape vulnerabilities stem from forgotten Unix fundamentals."
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
    alt: "Linux process tree showing container isolation"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

With Kubernetes everywhere, it is easy to forget what is actually happening under the hood. We write YAML manifests with `securityContext` blocks, set `runAsUser: 1000`, and move on. But what does that actually mean? What is a container, really?

This article is the second in my "Back to Basics" series. In December, I wrote about [SSH hardening configuration](https://www.msbiro.net/posts/back-to-basics-sshd-hardening/). This piece continues that journey.

**Containers are just Linux processes with extra isolation.** Everything Kubernetes does with permissions, users, and security contexts is built on top of concepts that have existed in Unix systems for decades. Understanding these basics makes container security much less mysterious.

---

## Part 1: The Unix Foundation

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

## Part 2: Applying These Concepts to Containers and Kubernetes

Now that we understand Unix permissions and process identity, let us see how containers use these features and how Kubernetes `securityContext` maps directly to them.

### What Is a Container?

A container is a process (or process group) with Linux kernel isolation features:

1. **Namespaces**: Isolate what processes see (PID, network, filesystem)
2. **Cgroups**: Limit resource usage (CPU, memory, I/O)
3. **Capabilities**: Fine-grained privileges instead of all-or-nothing root
4. **Seccomp**: Filter system calls
5. **AppArmor/SELinux**: Mandatory access controls

It is still just a process.

On the host, containers appear as regular processes:

```bash
$ ps aux | grep nginx
root      12345  0.0  0.1  12345  6789 ?  Ss  10:00 nginx: master process
www-data  12346  0.0  0.0  12346  6790 ?  S   10:00 nginx: worker process
```

That nginx container is PID 12345 on the host. It appears as PID 1 inside the container because of PID namespace isolation.

When `kubectl exec` fails, inspect containers using standard Linux tools:

```bash
# View namespaces
ls -la /proc/12345/ns/

# Check cgroups
cat /proc/12345/cgroup

# View UIDs
cat /proc/12345/status | grep Uid
```

### Kubernetes securityContext Decoded

Every Kubernetes `securityContext` field maps to a Linux feature.

**runAsUser: Setting the UID**

```yaml
securityContext:
  runAsUser: 1000
```

Equivalent to:
```bash
su -c "myapp" -s /bin/sh user1000
```

Or in Dockerfile:
```dockerfile
USER 1000
```

Regular users start at UID 1000. Using non-zero UIDs prevents accidental host user mapping.

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

Kubernetes runs `chown -R 2000:2000` on the volume mount point. This lets the container user (UID 1000) access files via group ownership (GID 2000).

**Limitation**: Only works on volumes supporting ownership changes (emptyDir, configMap, secret). HostPath and NFS may ignore this.

### Container Escape Risks

Understanding Unix helps you understand container escapes.

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

When a container runs as UID 0 and escapes (via kernel bugs, misconfigured volumes, or privileged mode), it becomes root on the host.

**Rule**: Never run containers as root. Use `runAsUser: 1000` everywhere.

### Debugging Containers Using Unix Tools

When troubleshooting, use Linux fundamentals.

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

Kubernetes 1.33+ enables user namespace isolation by default. I previously covered how [user namespaces remap container root to unprivileged host users](/posts/kubernetes-133-user-namespace-isolation-security-matters/). Instead of preventing escapes, user namespaces make them harmless.

---

## Conclusion

Containers are not magic or virtual machines. They are **Linux processes with isolation**.

Kubernetes security settings map directly to Unix features:

| Kubernetes | Linux Equivalent |
|------------|------------------|
| `runAsUser: 1000` | Process UID |
| `allowPrivilegeEscalation: false` | Blocks setuid |
| `readOnlyRootFilesystem: true` | Read-only mount |
| `capabilities` | Fine-grained privileges |
| `privileged: true` | Broken isolation |

When YAML abstractions fail, debug at the Linux level. Understanding these foundations helps you understand attack vectors and build secure systems.

Master the basics: keep containers unprivileged, minimize setuid binaries, remember it is all just processes, users, and permissions.

---

## Further Reading

- [Linux Capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Namespaces in Operation](https://lwn.net/Articles/531114/)
- [Kubernetes Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [Podman Rootless](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
