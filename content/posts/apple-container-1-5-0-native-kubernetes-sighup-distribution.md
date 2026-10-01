---
title: "Apple Container 1.5.0: Native Kubernetes for SIGHUP Distribution"
date: 2026-10-01T12:00:00+01:00
tags: [
  "apple-container", "kubernetes", "kubeadm", "devops",
  "apple-silicon", "sighup-distribution", "furyctl", "cloud-native"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Apple Container 1.5.0 creates the Kubernetes cluster natively, fixes the retained kernel failure, and runs SIGHUP Distribution without manual kubeadm recovery."
canonicalURL: "https://www.msbiro.net/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/"
disableShare: true
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "Apple Container 1.5.0 native Kubernetes and SIGHUP Distribution"
    caption: "Apple Container Kubernetes on the recommended Kata kernel"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

My first [Apple Container Kubernetes and SIGHUP Distribution lab](/posts/apple-container-local-kubernetes-sighup-distribution/) needed a deliberately narrow recovery path. `container k8s create` started the node but failed during preparation, so the lab helper applied two TCP MSS rules through legacy iptables and completed the kubeadm bootstrap itself.

Apple Container 1.5.0 closes that story. It fixed the retained kernel failure at the source, so the cluster is created natively and the SIGHUP Distribution installs on top without any bootstrap workaround. The kernel backstory still matters, because the upgrade behavior that caused the failure is unchanged, and the remediation is the same explicit command.

This article documents the failure, the kernel upgrade path, and a clean SIGHUP Distribution installation on Container 1.5.0 with Furyctl v0.35.1 and Distribution v1.35.1.

---

## The upgrade retained the old kernel

After updating to Container 1.4.1, I first repeated the native cluster creation command against the existing kernel:

```bash
container k8s create --name sighup-iptables-141-repro --cpus 2 --memory 4g
```

It failed with the same signature reported in [apple/container#2120](https://github.com/apple/container/issues/2120):

```text
Error: node prep failed on sighup-iptables-141-repro: net.ipv4.ip_forward = 1
registry.k8s.io/pause:3.10.1
```

The message is misleading. The `net.ipv4.ip_forward` setting succeeds, then the final node-preparation step invokes `iptables-nft` and fails before its stderr is surfaced. The relevant difference was not the Container 1.4.1 binary. This host still had `vmlinux-6.12.28-153`, retained from the pre-1.0 installation.

The current upstream discussion records the same upgrade path. Existing Container kernels are kept when the runtime starts, so an application upgrade alone can leave an old guest kernel active. That mattered here because the Kubernetes plugin invoked `iptables-nft` during node preparation.

Container 1.5.0 removed that assumption (see the fix below), so a retained kernel no longer aborts cluster creation. The upgrade behavior itself is unchanged, and the recommended kernel remains the supported path.

---

## Install the recommended kernel deliberately

Container provides the appropriate explicit command:

```bash
container system kernel set --recommended --force
```

On this Mac, it installed Kata 3.32.0 and `vmlinux-6.18.35-197-debug`. The `--force` flag is important. It replaces the selected default kernel, so it should be a conscious choice on a host that may use a custom kernel.

After removing the old test node, I created the real local cluster with the allocation used by the SIGHUP local profile:

```bash
container k8s create --name sighup-local --cpus 6 --memory 16g
```

On Container 1.5.0 the native creation completed in about a minute. The node reported Kubernetes v1.35.5 and kernel `6.18.35`. No helper injected iptables rules, ran kubeadm, or applied kindnet manually.

Node preparation now uses the node's configured `iptables` backend instead of hard-coding `iptables-nft` ([pull request #2145](https://github.com/apple/container/pull/2145)), and `iptables -t mangle -S` listed both TCP MSS rules successfully.

I recorded the corrected analysis and a suggestion to surface retained old kernels during upgrade in my [follow-up comment on #2120](https://github.com/apple/container/issues/2120#issuecomment-5634405404).

---

## Deploying SIGHUP Distribution on the clean cluster

The cluster creation workaround disappeared, but the SIGHUP Distribution setup still has two local requirements.

Apple Container's Kubernetes plugin does not install a default StorageClass. I installed Rancher's local-path provisioner and marked it as default:

```bash
./scripts/install-local-path-storage.sh
kubectl get storageclass
```

The local profile also needs more inotify capacity for Fluent Bit than the node default provides:

```bash
./scripts/configure-node-sysctls.sh
```

The helper persists and applies the existing SIGHUP values:

```text
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=524288
```

I then applied SIGHUP Distribution v1.35.1 with Furyctl v0.35.1:

```bash
./scripts/install-furyctl.sh
export FURYCTL_BIN="$PWD/.tools/furyctl/furyctl"
./scripts/deploy-sighup-distribution.sh
```

Furyctl completed successfully. The local profile kept Apple Container's kindnet CNI and deployed HAProxy ingress, cert-manager, Loki, Prometheus, Tempo, Forecastle, and the required storage. It still removes the two systemd-only host tailers after the logging module is rendered because that limitation belongs to the node environment, not to the old kernel issue.

One Apple silicon detail surfaced in the tooling around the installation. `furyctl apply` worked, but `furyctl get cluster-info` failed with `bad CPU type in executable`. The installed furyctl was the correct arm64 build.

Its tool cache dated back to September and contained x86_64 mise, kubectl, helm, kustomize, kapp, and yq. The source was a leftover x86_64 furyctl copy in `/usr/local/bin`, installed months earlier through the furyctl docs snippet that defaults to amd64. It had populated the shared cache while Rosetta was still available, and the arm64 furyctl kept reusing those binaries instead of downloading native ones.

Clearing the cache and running `furyctl download dependencies` fetched native arm64 tools, and the verification command then reported the cluster correctly.

---

## Evidence from the native run

The table below summarizes the checks from that run.

| Area | Test | Observed result |
| --- | --- | --- |
| Kernel upgrade | Installed the recommended default with `container system kernel set --recommended --force` | Kata 3.32.0, kernel `6.18.35` |
| Native Kubernetes | Created `sighup-local` with 6 CPUs and 16 GB on Container 1.5.0 | Completed in about a minute, node Ready on Kubernetes v1.35.5 |
| Node preparation | Listed the mangle table through the configured `iptables` backend | Both TCP MSS rules were present |
| Storage | Installed local-path and applied the SIGHUP profile | The default StorageClass was present and all distribution PVCs were Bound |
| Logging | Applied the inotify settings and removed systemd-only tailers | Fluent Bit was Ready with no restarts |
| SIGHUP Distribution | Ran Furyctl v0.35.1, Distribution v1.35.1 | Apply succeeded, Grafana, Fluent Bit, and HAProxy rollouts completed |
| furyctl tools | Cleared a stale x86_64 tool cache, ran `download dependencies` | All tools arm64, `get cluster-info` reported the cluster |
| Monitoring access | Port-forwarded Grafana and Forecastle | Both returned HTTP 200 |
| Local image flow | Built, tagged, loaded, and deployed a local NGINX image | The demo service returned the expected page through a port-forward |

The complete, reproducible material is in the [fury-apple-container repository](https://github.com/matteobisi/fury-apple-container). The repository now starts with the native flow and keeps the Container 1.2.2 manual recovery under `legacy/container-1.2.2-iptables-recovery/`. The older scripts and journal remain available for analysis, but they are not part of a new installation.

---

## Restart means recreate, for now

Container 1.5.0 also removed `container k8s start`. A stopped node cannot be restarted in place. The only path back is deleting the cluster and creating it again, which discards all node-local data, including the SIGHUP installation.

The removal is temporary. The maintainers took the command out in [pull request #2290](https://github.com/apple/container/pull/2290) because restarting had accumulated bugs (see [issue #2156](https://github.com/apple/container/issues/2156)) and the user experience of the experimental plugin still needs a rethink.

Suspending the command keeps the code easier to reason about and removes a known way to break a cluster while the plugin itself improves. They explicitly asked for restart use cases and feedback on the decision, so this is a good moment to report yours if stopping a local cluster matters to your workflow.

---

## The current local workflow

For an Apple-silicon Mac running Container 1.5.0, the working sequence is:

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

The profile is still a single-node local environment. Its local-path volumes are node-local, it does not provide a production service load balancer, and ClusterIP services are accessed from macOS through `kubectl port-forward`. These limits make it unsuitable for production, but they do not prevent a complete local SIGHUP Distribution test environment.

The important change is smaller than the earlier workaround: a current Container release needs its current recommended guest kernel. Once that condition is met, Apple Container creates the Kubernetes cluster normally and the SIGHUP setup can focus on the application-level constraints that actually remain.

---

## Sources

- [Apple Container 1.5.0 release](https://github.com/apple/container/releases/tag/1.5.0)
- [Node-preparation iptables backend fix, #2145](https://github.com/apple/container/pull/2145)
- [Temporary removal of `k8s start`, #2290](https://github.com/apple/container/pull/2290)
- [Cluster restart issues, #2156](https://github.com/apple/container/issues/2156)
- [Apple Container Kubernetes node-preparation failure, #2120](https://github.com/apple/container/issues/2120)
- [Follow-up reproduction and kernel-upgrade result](https://github.com/apple/container/issues/2120#issuecomment-5634405404)
- [Original Apple Container Kubernetes and SIGHUP Distribution lab](/posts/apple-container-local-kubernetes-sighup-distribution/)
- [SIGHUP Distribution on Minikube](https://docs.sighup.io/docs/getting-started/distro-on-minikube)
- [fury-apple-container lab repository](https://github.com/matteobisi/fury-apple-container)
