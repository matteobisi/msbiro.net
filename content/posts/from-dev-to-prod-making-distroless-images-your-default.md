---
title: "From Dev to Prod: Making Distroless Images Your Default "
date: 2025-06-17T10:10:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "distroless", "containers", "cdebug", "docker debug", "container security",
  "devsecops", "kubernetes debugging", "ephemeral containers", "multi-stage builds", "container runtime"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Explore the importance of using distroless container images to reduce security vulnerabilities in production. This post covers practical advice on adopting distroless images using multi-stage builds, along with comprehensive debugging techniques including the open-source cdebug tool, Docker Debug, and Kubernetes' kubectl debug with ephemeral containers. Learn how strategic container image choices improve security, efficiency, and maintainability from development through production."
canonicalURL: "https://www.msbiro.net/posts/from-dev-to-prod-making-distroless-images-your-default/"
disableHLJS: true # to disable highlightjs
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
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
**Security should be a primary driver in IT!**   
   
Everyone understands the importance of running secure, reliable code at every level of our infrastructure. Since the container revolution began a decade ago with Kubernetes 1.0, traditional IT administrators have lost some control to developers, who can now use Dockerfiles to package and deploy software at unprecedented speed.  
  
**But at what cost?**

As organizations adopted runtime security tools to monitor containers and processes, it quickly became clear that pulling base images from public repositories often introduced a flood of vulnerabilities.  
  
**Developers cannot, and should not, bear all the responsibility alone.**   
  
Modern security teams must enforce policies that help sysadmins maintain order and reduce risk. In 2025, simply pulling the latest image of your preferred distro from Docker Hub and running it in production is no longer acceptable. With great power comes great responsibility: we must use that power wisely.

Let’s clarify why distroless images matter:

- Containers are built from base images.
- The fewer packages in the base image, the fewer vulnerabilities over time.
- Fewer vulnerabilities mean fewer problems for your organization.

Isn’t that straightforward?

## **Distroless Images: Why and How**

At the recent DevSecOps Days 2024 in Bologna, my colleague Michele Buccarello and I led a session on the [importance of distroless images](https://2024.devsecopsday.it/talks_speakers/#a-journey-to-distroless-3). Developers asked us how to use distroless images in production and how to debug their software effectively.

First, no one is forcing you to use distroless images in your development environment. During development, having a shell and debugging tools inside containers is often necessary. However, for quality assurance and production, you should “strip” unnecessary binaries and libraries using [multi-stage builds](https://docs.docker.com/build/building/multi-stage/), providing only the minimal environment needed for your application.

Debugging a distroless container, even on your laptop, is possible! Here are some practical tools and approaches:

## **cdebug**

The open-source tool cdebug, available since 2022, describes itself as the **“Swiss Army knife of container debugging”**.   
Install it with:

```
brew install cdebug
```

To debug a distroless container, run:

```
cdebug exec -it nginx-distroless
```

You’ll get a prompt like this:

```
/ #
```

From here, you can perform standard debugging tasks inside the container. Magic? Almost.

What cdebug actually does is launch another container (you can verify this with `podman ps`) that attaches to the process and network namespaces of your target container. By default, it uses a BusyBox sidecar, but you can specify your preferred tool if needed.

For more examples, [consult the cdebug GitHub repository](https://github.com/iximiuz/cdebug).

## **Docker Debug**

If you’re a Docker Pro or Business user, Docker offers its own tool: `docker debug`. Like cdebug, it launches a sidecar container connected to your target container for debugging. This custom container includes editors, curl, and other tools, and you can add more with the install command. See [Docker’s official documentation](https://docs.docker.com/reference/cli/docker/debug/) for details.

## **kubectl debug**

What if you’re running on Kubernetes? The ephemeral containers feature has been generally available since Kubernetes 1.23 (originally introduced as alpha in 1.18). This allows you to debug running pods without extra feature gates.

For example, to launch a BusyBox container connected to the “mypod” pod:

```
kubectl debug mypod -it --image=busybox --target=myapp-container
```

You can use `kubectl debug` in several ways:

- **Workload:** Create a copy of an existing pod with modified attributes (e.g., a new image version).
- **Ephemeral Container:** Add a temporary container to a running pod for debugging, without restarting it.
- **Node:** Launch a pod in the node’s host namespaces to access the node’s filesystem.

Refer to the [official Kubernetes documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/) for more information.

## **Conclusions**

A strategic, coordinated approach to base images is essential for reducing the attack surface of your workloads—wherever they run. Using minimalistic images brings long-term benefits in:

- Security
- Storage efficiency
- Reduced operational overhead

In 2025, developers have plenty of tools and methods to work securely and efficiently. Remember to schedule regular rebuilds of your containers, even if they appear to be running smoothly.  
**New CVEs are discovered over time; what is secure today may be vulnerable tomorrow.** By rebuilding and updating containers regularly, you keep dependencies fresh and make future upgrades easier.

If you aren't already using distroless images regularly, I hope you’re now thinking: “Okay, where can I find them?”

Of course, there are two ways: you can build them yourself (or have your developers do it), or you can rely on a catalog. In 2025, there are plenty of options, such as:

- [Chainguard](https://www.chainguard.dev/)
- [Docker](https://www.docker.com/products/hardened-images/)

Personally, I have tried and explored only the Chainguard solution, and **I can say they are VERY good**, I really love what they have built!!

Usually, these kinds of solutions take a “by image” approach, but my advice is to investigate which images your company uses most, identify which ones have the most CVEs, and start with the three that cover the majority.
Once the adoption of hardened images is complete, I’m sure you’ll also find the budget for more, especially after comparing the list of active CVEs.

Stay secure, stay efficient: **embrace distroless images in production!**
