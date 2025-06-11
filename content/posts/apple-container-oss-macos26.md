---
title: "Apple container announced"
date: 2025-06-10T14:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["apple","container","container runtime","macos26"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#description: "Desc Text."
canonicalURL: "https://www.msbiro.net/posts/apple-container-oss-macos26/"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
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
As you probably know, [Apple is running WWDC 25](https://developer.apple.com/wwdc25/), and yesterday there were a lot of exciting announcements.
Among these, aside from the OS updates, Apple announced "container" and containerization support for macOS 26.

**Here are the key features:**

- Manage OCI images
- Interact with remote registries
- Create and populate ext4 file systems
- Interact with the Netlink socket family
- Create an optimized Linux kernel for fast boot times
- Spawn lightweight virtual machines
- Manage the runtime environment of virtual machines
- Spawn and interact with containerized processes
- Use Rosetta 2 for executing x86_64 processes on Apple silicon

In fact, the "container" client will be able to spawn a lightweight VM with an **optimized Linux kernel and small rootFS**, where you can run Linux containers using Rosetta 2 for executing x86 instructions.   
The interesting part from a security perspective is that every container will run isolated inside its own lightweight VM.  
  
This announcement is **interesting** and could be a **valuable alternative** for developers compared to the usual Docker Desktop (which is free only for personal use) and Podman/Podman Desktop (free and open source).

![Apple container](landing-movie.gif)

**Software requirements:**

- macOS 15 or newer and Xcode 26 Beta
- macOS 26 Beta 1 or newer

**Hardware requirement:**

- Apple silicon CPU

You can find the two related GitHub repositories below, with license Apache 2.0:

- [containerization](https://github.com/apple/containerization)
- [container](https://github.com/apple/container)