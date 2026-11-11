---
title: "Apple Container 1.7.1: A Short Follow-Up on the SIGHUP Distribution Lab"
date: 2026-11-10T09:00:00Z
tags: ["linux", "kubernetes", "devsecops", "cloud-native"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Apple Container maintainers keep changing the Kubernetes CLI. The SIGHUP Distribution lab still works on 1.7.1, but the commands and a few habits need to change."
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
    alt: "Apple Container 1.7.1 follow-up on the SIGHUP Distribution lab"
    caption: "Apple Container 1.7.1 and SIGHUP Distribution v1.36.0"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In October I published a [follow-up](/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/) confirming that Apple Container 1.5.0 creates the Kubernetes cluster natively and that SIGHUP Distribution installs on top of it. That post was verified with Container 1.5.0, Kubernetes v1.35.5, Furyctl v0.35.1, and Distribution v1.35.1.

Container is now at 1.7.1, and the CLI has moved again. This is another short follow-up, not a walkthrough. The Kubernetes plugin is still experimental. The maintainers keep reshaping its commands, and some of what I documented a few weeks ago no longer matches the tool. The approach works, though, and the full reproducible material lives in the [fury-apple-container repository](https://github.com/matteobisi/fury-apple-container) if you want to run it end to end. Everything below comes from a fresh run on 1.7.1.

---

## The CLI is still moving

The command namespace changed in 1.6.1. `container k8s` is now `container cluster`, with no alias, so every command in my [first](/posts/apple-container-local-kubernetes-sighup-distribution/) and [second](/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/) lab posts returns an error:

```text
$ container k8s --help
Error: unknown command 'k8s'
```

Alongside the rename, the plugin changed shape several times:

| Change | Release |
| --- | --- |
| Multi-worker clusters, and `--cni NONE` to skip CNI installation | 1.6.0 |
| `load-image` removed in favour of `--shared-image-store` at creation | 1.7.1 |
| `cluster list` shows one row per cluster, `--nodes` for the per-node view | 1.7.1 |
| Container lifecycle commands refuse cluster nodes | 1.7.1 |
| Per-group worker sizing with `--workers count=,cpus=,memory=` | 1.7.1 |

The practical effect is smaller than the list suggests. The lab scripts needed the command renames, and the local image workflow lost an entire step: `--shared-image-store` lets cluster nodes read Apple's local image store directly. There is no import command and no fully qualified name requirement any more. I deployed a bare `shortname-demo:dev` with `imagePullPolicy: IfNotPresent` and it started, where the earlier releases refused the equivalent bare name.

The lifecycle restriction is worth knowing before you go looking for a cluster. `container start sighup-local` now returns `container with ID sighup-local not found`, and nodes no longer appear in `container list`. Use `container cluster ls --nodes` instead. Node-level `container exec` still works, which matters because the lab needs it to apply the inotify settings for Fluent Bit.

---

## Still working, with three details to watch

The rerun used Kubernetes v1.36.4, SIGHUP Distribution v1.36.0, and Furyctl 0.36.1. Furyctl applied the local profile and reported success. All 58 pods were running with no restarts, all 16 PVCs were bound through the default `local-path` class, and both Grafana and Forecastle answered HTTP 200 through a port-forward.

Three details need attention. Two are old friends; the third is new.

The recommended guest kernel does not follow an application upgrade. My host was still on the 1.5.0 kernel, `vmlinux-6.18.35-197-debug`, while 1.7.1 recommends Kata 4.3.1 on `vmlinux-6.18.56-208-debug`. The new kernel was not even on disk. `container system kernel set --recommended --force` fixes it, and this is the second time the same behaviour has shown up across this series.

Apple Container still creates no default StorageClass, so Rancher's local-path provisioner remains a required step. The node image now preloads the provisioner images without deploying them, which is a useful hint and not yet a fix.

There is also a kubeconfig detail I hit while testing multi-worker clusters. `container cluster create` writes to the file named by `$KUBECONFIG`, not only to `~/.kube/config`, and deleting a cluster does not clean up contexts it created elsewhere. If your lab exports `KUBECONFIG` early, rewrite the file with `container cluster write-config` and reselect the context after any cluster churn. Otherwise `kubectl` keeps dialling a port that no longer exists, or fails against the certificate authority of a cluster you already deleted.

---

## ACTUI keeps pace

[ACTUI](https://github.com/matteobisi/apple-container-tui) is a keyboard-first terminal UI for Apple Container. I built it for pleasure: I did not want to keep looking up the CLI syntax, and I wanted to play with spec-driven development on something real. A TUI that shells out to a CLI inherits every breaking change in it, so the 1.7.1 release was work for that project too.

That is where [GitHub spec-kit](https://github.com/github/spec-kit) and my personal Command Code GOAT subscription come in. Each Apple Container release becomes a spec, the spec drives the plan and the tasks, and the implementation lands against a documented contract instead of a mental note. The workflow is why the TUI kept pace with each of these Container releases without losing the command previews, the dry-run mode, or the safe delete confirmations along the way.

On the Kubernetes side, ACTUI now renders the one-row-per-cluster list with a per-node detail toggle, and it offers no `load-image` action because the command no longer exists. Cluster creation exposes the shared image store, node image, CNI choice, and per-group worker sizing.

---

## Where to go for the full walkthrough

I am not republishing the whole guide here. The complete, verified sequence, the profile, the scripts, and the evidence journal are in the [fury-apple-container repository](https://github.com/matteobisi/fury-apple-container):

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

If you are following my earlier posts as written, replace `container k8s` with `container cluster` throughout and drop the `load-image` step in favour of `--shared-image-store`. Pin the node image to `kindest/node:v1.36.4` for Distribution v1.36.0, and expect Kata 4.3.1 as the recommended kernel. The structure of the walkthrough holds up, which says something about the approach even if the commands keep moving.

The plugin remains experimental. Each release removes a workaround I documented in the previous one, and each release adds a new detail to track. It works well enough for a disposable local evaluation environment, and I would still point someone to minikube first if they only need a cluster up quickly.

---

## Sources

- [Apple Container 1.7.1 release](https://github.com/apple/container/releases/tag/1.7.1)
- [Apple Container 1.6.1 release, rename to `container cluster`](https://github.com/apple/container/releases/tag/1.6.1)
- [Shared image store pull request, #2437](https://github.com/apple/container/pull/2437)
- [Lifecycle commands exclude cluster nodes, #2462](https://github.com/apple/container/pull/2462)
- [Apple Container Kubernetes node-preparation failure, #2120](https://github.com/apple/container/issues/2120)
- [SIGHUP Distribution v1.36.0 release](https://github.com/sighupio/distribution/releases/tag/v1.36.0)
- [fury-apple-container lab repository](https://github.com/matteobisi/fury-apple-container)
- [Apple Container TUI (ACTUI)](https://github.com/matteobisi/apple-container-tui)
- [First lab post: Apple Container Kubernetes on macOS](/posts/apple-container-local-kubernetes-sighup-distribution/)
- [Second post: Apple Container 1.5.0 native Kubernetes](/posts/apple-container-1-5-0-native-kubernetes-sighup-distribution/)
