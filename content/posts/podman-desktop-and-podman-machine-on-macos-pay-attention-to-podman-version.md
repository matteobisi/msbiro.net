---
title: "macOS, Podman Desktop and the Podman Machine: Pay Close Attention to the Podman Version"
date: 2025-01-10T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "podman", "podman desktop", "podman machine",
  "container", "container runtime", "macos",
  "virtualization", "container management",
  "compatibility", "troubleshooting", "developer tools"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post explores compatibility issues when using Podman Desktop on macOS, particularly the Podman machine failing to start due to legacy Podman installations. It shares practical troubleshooting steps including identifying version conflicts, removing unsupported Podman machines, and successfully recreating them for smooth container management. Essential reading for developers managing container runtimes on macOS."
canonicalURL: "https://www.msbiro.net/posts/podman-desktop-and-podman-machine-on-macos-pay-attention-to-podman-version/"
disableHLJS: true # to disable highlightjs
disableShare: true
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
Using [Podman](http://www.podman.io/) as the standard tool requested by clients for running local containers outside of a Kubernetes environment, I decided to start the year by installing Podman Desktop on my company MacBook.

Podman Desktop features a user interface (UI) similar to Docker Desktop, making it easier to manage containers and images.   
It also includes plugin management to extend its functionality, such as deploying containers on Kubernetes.

After installing Podman Desktop version 1.15.0, I proceeded with the setup but encountered issues with the Podman machine (the virtual machine dedicated to running containers) which failed to start. There were no errors; it just hung during startup.

After performing all the necessary checks and finding no logs, I tried the usual troubleshooting steps, including cleaning up and reinstalling. This resolved the issue and revealed the cause: my MacBook previously had an older version of Podman installed, which **I had completely forgotten about**.



Following the new installation of Podman Desktop, since no existing version was detected, the setup prompted me to install a newer version. At this point, the setup for Podman Desktop identified that the existing Podman machine was incompatible with the current release.

![podman machine error](podman-machine-unsupported.png)

After confirming the removal of the unsupported Podman machine and proceeding with its recreation, the Podman machine started successfully.

![podman hello world](podman-hello-world.png)

**Key Takeaway:**
When using Podman Desktop on macOS, especially after previous installations or upgrades, ensure that the Podman machine is compatible with your current Podman version. If you encounter startup issues, check for legacy Podman installations or machines that may cause version conflicts. Removing and recreating the Podman machine often resolves these compatibility problems.