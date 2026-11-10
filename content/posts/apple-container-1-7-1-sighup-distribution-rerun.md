---
title: "Apple Container 1.7.1: Rerunning the SIGHUP Distribution Lab After Four Releases"
date: 2026-11-10T09:00:00Z
tags: ["linux", "kubernetes", "devsecops", "cloud-native"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Apple Container renamed its Kubernetes plugin, removed load-image, and moved the recommended kernel again. I reran the full SIGHUP Distribution lab on 1.7.1 to see what still holds."
canonicalURL: "https://www.msbiro.net/posts/apple-container-1-7-1-sighup-distribution-rerun/"
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
    alt: "Apple Container 1.7.1 and SIGHUP Distribution rerun"
    caption: "Apple Container 1.7.1 Kubernetes and SIGHUP Distribution v1.36.0"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

My colleagues published [How to test SIGHUP Distribution locally with Apple's container Kubernetes Plugin](https://blog.sighup.io/how-to-test-sighup-distribution-locally-with-apples-container-kubernetes-plugin/) on 9 November 2026. I wrote that walkthrough, and it was verified with Container 1.4.1, Kubernetes v1.35.5, Furyctl v0.35.1, and SIGHUP Distribution v1.35.1.

By the time it went live, Container was at 1.7.1. Four releases landed between the verification and the publication, and every one of them touched the Kubernetes plugin. The command namespace changed, a subcommand disappeared, the recommended guest kernel moved, and the way locally built images reach a cluster node was replaced.

That makes the published article outdated in places, and it makes my [first](https://www.msbiro.net/posts/apple-container-local-kubernetes-sighup-distribution/) and [second](https://www.msbiro.net/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/) lab posts outdated too. Rather than patch three documents from memory, I rebuilt the whole lab on Container 1.7.1 with SIGHUP Distribution v1.36.0, and this is the verified result.

Everything below comes from a single run on an Apple silicon Mac with macOS 27.2, 10 CPUs, and 32 GB of memory. Nothing here is inferred from release notes alone.

---

## What changed in the plugin

The rename is the part that breaks existing scripts immediately. `container k8s` became `container cluster` in 1.6.1. There is no alias, so every command in the published walkthrough and in both of my posts returns an error:

```text
$ container k8s --help
Error: unknown command 'k8s'
```

The current surface is smaller than the old one:

```text
$ container cluster --help
SUBCOMMANDS:
  create         Create and start a local Kubernetes cluster and optional worker nodes
  delete, rm     Delete a Kubernetes cluster and its nodes
  list, ls       List clusters
  write-config   Write the cluster context to a Kubernetes configuration file
```

The rest of the changes are summarized below, with the release that introduced each one.

| Change | Release | Effect on the lab |
| --- | --- | --- |
| `container k8s` renamed to `container cluster` | 1.6.1 | Every script and every published command needed rewriting |
| Multi-worker clusters, `--cni NONE` | 1.6.0 | New capability, nothing in the lab depended on it |
| `load-image` removed, replaced by `--shared-image-store` on create | 1.7.1 | The local image workflow lost its import step |
| `cluster list` shows one row per cluster, `--nodes` for the per-node view | 1.7.1 | Parsing and documentation changed |
| Container lifecycle commands refuse cluster nodes | 1.7.1 | `container start` on a node now fails |
| Workers can be sized per group with `--workers count=,cpus=,memory=` | 1.7.1 | New capability |
| Recommended kernel moves to Kata 4.3.1, `vmlinux-6.18.56-208-debug` | 1.7.1 | Kernel remediation needed again |

The `start` removal I documented in the 1.5.0 post still stands. There is no `container cluster start` either, so a stopped node is deleted and recreated, not resumed.

---

## The kernel moved again, and the host did not

The 1.5.0 post ended with a recommendation: run `container system kernel set --recommended --force` deliberately, because an application upgrade does not move the selected guest kernel forward.

That is still true, and it applied again. Before I changed anything for this run, the host was still on the kernel the 1.5.0 lab installed:

```text
$ readlink "$HOME/Library/Application Support/com.apple.container/kernels/default.kernel-arm64"
.../kernels/vmlinux-6.18.35-197-debug
```

The `kernels/` directory held only `vmlinux-6.12.28-153` and `vmlinux-6.18.35-197-debug`. The 1.7.1 recommended kernel, Kata 4.3.1 on `vmlinux-6.18.56-208-debug`, was not on disk at all, and a cluster created in that state reported kernel `6.18.35`.

The remediation is the same command, with a different result:

```bash
container system kernel set --recommended --force
```

It downloaded Kata 4.3.1 and created `vmlinux-6.18.56-208-debug` together with a new `default.kernel-arm64` symlink. A node created afterwards reported `6.18.56`. The `--force` flag still replaces the selected default, so it remains a conscious choice on a host that depends on a custom kernel.

Two upgrades, the same retention behaviour, two different kernel versions. If there is one habit worth keeping from this series, it is to check the effective default kernel after every Container upgrade rather than assuming the recommended one is in use.

---

## Rebuilding the lab on 1.7.1

I deleted the orphaned node from the 1.5.0 lab first. It had an interesting symptom: it appeared in `container list` but not in `container cluster list`, because it was created before the rename and the cluster metadata no longer matched. `container delete` removed it without complaint.

The cluster creation itself needed no workaround:

```bash
container cluster create \
  --name sighup-local \
  --cpus 6 \
  --memory 16g \
  --node-image docker.io/kindest/node:v1.36.4 \
  --shared-image-store
```

It completed in 40 seconds. The node came up Ready on Kubernetes v1.36.4 with containerd 2.3.4 and kernel `6.18.56`.

The node image is pinned on purpose. Apple still defaults to `kindest/node:v1.35.5`, but SIGHUP Distribution v1.36.0 supports Kubernetes 1.36, so pinning `v1.36.4` keeps the cluster and the distribution on the same minor version. The `--node-image` flag is not new, it existed back in 1.5.0, but it matters more now that the distribution has moved to 1.36.

One gotcha cost me a minute. The `.state/sighup-local.kubeconfig` left over from the September lab still pointed at the deleted cluster's certificate authority:

```text
Unable to connect to the server: tls: failed to verify certificate:
x509: certificate signed by unknown authority
```

Rewriting it fixed the problem, and any script that reuses a fixed kubeconfig path should do the same after recreating a cluster:

```bash
container cluster write-config --name sighup-local \
  --kubeconfig "$PWD/.state/sighup-local.kubeconfig"
kubectl config use-context sighup-local
```

A second kubeconfig trap appeared later, while I was testing multi-worker clusters. `container cluster create` writes the new context to the file named by `$KUBECONFIG`, not only to `~/.kube/config`. Because the lab variable was exported, creating a throwaway cluster merged its context into the lab kubeconfig and selected it. Deleting the cluster removed it from `~/.kube/config` but left the dangling context behind, still marked current, so `kubectl` kept dialling a port that no longer existed. The same two commands above clear it.

That behaviour is worth a note in any lab guide that exports `KUBECONFIG` early: creation and deletion do not fully own the file they touch.

The new listing output is worth showing, because it changed shape:

```text
$ container cluster ls
NAME          NODES  K8S VERSION  STATE
sighup-local  1      v1.36.4      running

$ container cluster ls --nodes
CLUSTER       NODE          ROLE                  STATE    CPUS  MEMORY  ADDR          PORTS
sighup-local  sighup-local  control-plane,worker  running  6     16 GB   192.168.64.5  6445->6443
```

The plugin still publishes only the Kubernetes API to macOS. There is no service load balancer, so `kubectl port-forward` remains the portable way to reach anything inside the cluster.

---

## The local image workflow got simpler

The old workflow required a fully qualified name, a separate `load-image` step, and `imagePullPolicy: Never`. It was documented in my first post with a specific failure case: a bare image name was rejected with `ErrImageNeverPull`.

On 1.7.1, `load-image` is gone. `--shared-image-store` at creation time lets cluster nodes read Apple's local image store directly, so the import step disappears:

```bash
container build -t docker.io/library/apple-container-local-demo:0.1.0 \
  --file demo/Containerfile demo
kubectl apply -f manifests/local-demo.yaml
```

The pod came up Ready with no registry access, and the service returned the expected page through a port-forward.

The fully qualified name requirement is gone as well. I tested it explicitly, with a bare tag and `imagePullPolicy: IfNotPresent`:

```bash
container build -t shortname-demo:dev --file demo/Containerfile demo
kubectl -n local-demo run shortname --image=shortname-demo:dev \
  --image-pull-policy=IfNotPresent --restart=Never
```

The pod started, and the event log said the image was already present on the machine and accessible to the pod. That removes a small but recurring source of confusion for anyone who followed the earlier posts.

---

## SIGHUP Distribution v1.36.0 on the new cluster

Two local requirements from the previous runs survived the version changes.

Apple Container still creates no default StorageClass, so Rancher's local-path provisioner is still required. The node image now preloads `local-path-provisioner` and `local-path-helper` images, but it does not deploy them, so the storage class still has to be installed and marked default explicitly.

The logging profile still needs more inotify capacity than the node provides. The defaults before tuning were:

```text
fs.inotify.max_user_instances = 128
fs.inotify.max_user_watches = 130278
```

The watches value scales with node memory, and a 16 GB node got a much higher default than the 4 GB node I probed, but `max_user_instances` stayed at 128 regardless of size. Since Fluent Bit needs an instance per watched pattern, the SIGHUP recommendation of `8192` instances and `524288` watches is still the right starting point.

The `container exec` path for applying them still works. That was my main concern about the 1.7.1 lifecycle restriction, and it turned out to be a non-issue: the filter covers `start`, `stop`, `kill`, and `delete`, not `exec`. What it does break is addressing a node as a container:

```text
$ container start sighup-local
Error: container with ID sighup-local not found
```

Cluster nodes are also absent from `container list` now, so `container cluster ls --nodes` is the only supported way to inspect them.

With storage and sysctls in place, Furyctl 0.36.1 applied the profile with SIGHUP Distribution v1.36.0 and reported success. The result:

| Area | Observed result |
| --- | --- |
| Cluster | One Ready node, Kubernetes v1.36.4, kernel `6.18.56`, containerd 2.3.4 |
| Distribution | v1.36.0 applied with Furyctl v0.36.1 |
| Workloads | 58 pods running, no restarts, across cert-manager, forecastle, ingress-haproxy, local-path-storage, logging, monitoring, tracing |
| Storage | 16 PVCs Bound through the default `local-path` class |
| Ingress | HAProxy ingress rolled out as a DaemonSet |
| Logging | Fluent Bit Ready with no restarts after the inotify tuning |
| Grafana | `http://127.0.0.1:3000/login` returned HTTP 200 |
| Forecastle | `http://127.0.0.1:18081/` returned HTTP 200 |
| Local image | Pod Ready from the shared image store, service returned HTTP 200 |

The deployment helper still removes the two systemd-only host tailers, `systemd-common` and `systemd-etcd`, because they cannot run inside a container node. That limitation belongs to the environment, not to the Container version, and I do not expect it to change.

---

## What still does not work

The honest list is shorter than it was, but it is not empty.

A stopped cluster cannot be restarted. There is no `container cluster start`, and a node's internal address is not stable across a stop and start, so recreating the cluster is the only supported path. That discards all node-local data, including the distribution.

There is still no service load balancer. `kubectl port-forward` is the way in.

There is still no default StorageClass. One extra script step, every time.

And there is a new, unrelated warning I hit on this run: the kubectl client on the Mac is 1.34.1 while the cluster server is 1.36.4, which exceeds the supported version skew. That is my client being old, not a Container problem, but it is worth checking before you interpret version-sensitive output.

My own sysctl and storage scripts remain necessary, which is a reasonable indicator that these are environment constraints rather than bugs waiting to be fixed.

---

## ACTUI now follows the 1.7.1 surface

I maintain [ACTUI](https://github.com/matteobisi/apple-container-tui), a keyboard-first terminal UI for Apple Container. It is a proof of concept, and it is also my main way of driving the CLI day to day.

Every Container release that changes the CLI breaks a TUI that shells out to it, so I updated ACTUI to 1.7.1 as well. The Kubernetes view follows the new command set:

- Cluster list renders the one-row-per-cluster shape, with a key to toggle the per-node detail from `cluster ls --nodes`
- No `load-image` action is offered anywhere, since the command no longer exists
- Cluster creation exposes the shared image store, the node image, the CNI choice, and per-group worker sizing
- Command previews show the exact 1.7.1 command before anything runs, and dry-run mode still blocks execution

The repository is at [github.com/matteobisi/apple-container-tui](https://github.com/matteobisi/apple-container-tui). It is a separate project from the SIGHUP lab, but the two are complementary: the lab is the reproducible evidence, and the TUI is the day-to-day interface.

---

## Corrections for the published SIGHUP article

If you are following the SIGHUP blog post as written, these are the commands and values that need to change for Container 1.6.1 and later. The structure of the walkthrough is still correct, which is a good sign, since the underlying approach survived four releases.

| Published | Current |
| --- | --- |
| `container k8s create` | `container cluster create` |
| `container k8s write-config` | `container cluster write-config` |
| `container k8s delete` | `container cluster delete` |
| `container k8s load-image` with a fully qualified name | `--shared-image-store` on `cluster create` |
| Kata 3.32.0, kernel `6.18.35` | Kata 4.3.1, `vmlinux-6.18.56-208-debug`, kernel `6.18.56` |
| `kindest/node:v1.35.5` | `kindest/node:v1.36.4` for Distribution v1.36.0 |
| Furyctl 0.35.1, Distribution v1.35.1 | Furyctl 0.36.1, Distribution v1.36.0 |
| local-path provisioner v0.0.34 | v0.0.37 |

Two prose corrections matter as much as the commands. The article says the kernel edge case affects hosts upgraded from a pre-1.0 installation; that is no longer the whole story, because the same retention now applies to a host upgraded from 1.5.0 to 1.7.1. And the image workflow paragraph that explains the fully qualified name requirement should be deleted, not edited, because the requirement is gone.

---

## The current workflow

For an Apple silicon Mac on Container 1.7.1, the sequence is unchanged in shape and different in detail:

```bash
git clone https://github.com/matteobisi/fury-apple-container.git
cd fury-apple-container
container system start
./scripts/install-recommended-kernel.sh
./scripts/create-cluster.sh
export KUBECONFIG="$PWD/.state/sighup-local.kubeconfig"
./scripts/configure-node-sysctls.sh
./scripts/install-local-path-storage.sh
./scripts/install-furyctl.sh
export FURYCTL_BIN="$PWD/.tools/furyctl/furyctl"
./scripts/deploy-sighup-distribution.sh
```

The [lab repository](https://github.com/matteobisi/fury-apple-container) is updated for 1.7.1, including the command renames, the shared image store, the pinned node image, and a release-by-release table of what changed. The verification journal records the evidence behind every row above, and the Container 1.2.2 recovery stays archived under `legacy/` as history.

The pattern across this series is consistent. Each Container release removes a workaround I documented in the previous one, which is a good trade. The two constants are the guest kernel not following an upgrade and the missing default StorageClass, and both are explicit, one-line fixes once you know about them.

---

## Sources

- [How to test SIGHUP Distribution locally with Apple's container Kubernetes Plugin (SIGHUP blog)](https://blog.sighup.io/how-to-test-sighup-distribution-locally-with-apples-container-kubernetes-plugin/)
- [Apple Container 1.7.1 release](https://github.com/apple/container/releases/tag/1.7.1)
- [Apple Container 1.6.1 release, rename to `container cluster`](https://github.com/apple/container/releases/tag/1.6.1)
- [Apple Container 1.6.0 release, multi-worker clusters](https://github.com/apple/container/releases/tag/1.6.0)
- [Shared image store pull request, #2437](https://github.com/apple/container/pull/2437)
- [Cluster list redesign pull request, #2451](https://github.com/apple/container/pull/2451)
- [Lifecycle commands exclude cluster nodes, #2462](https://github.com/apple/container/pull/2462)
- [Apple Container Kubernetes node-preparation failure, #2120](https://github.com/apple/container/issues/2120)
- [SIGHUP Distribution v1.36.0 release](https://github.com/sighupio/distribution/releases/tag/v1.36.0)
- [SIGHUP Distribution on Minikube](https://docs.sighup.io/docs/getting-started/distro-on-minikube)
- [fury-apple-container lab repository](https://github.com/matteobisi/fury-apple-container)
- [Apple Container TUI (ACTUI)](https://github.com/matteobisi/apple-container-tui)
- [First lab post: Apple Container Kubernetes on macOS](/posts/apple-container-local-kubernetes-sighup-distribution/)
- [Second post: Apple Container 1.5.0 native Kubernetes](/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/)
