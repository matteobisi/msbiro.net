---
title: "From Manual to GitOps: Simplifying Grafana Dashboard Configuration with Git Sync"
date: 2025-05-12T11:34:03+00:00
tags: [
  "grafana", "dashboard", "gitops",
  "git-sync", "observability-as-code", "ci-cd",
  "version-control", "grafana-12", "monitoring", "devops"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Starting with version 12, Grafana introduces the experimental Git Sync feature, enabling users to manage dashboards using a GitOps approach. This feature connects Grafana to a GitHub repository to synchronize dashboard JSON files, allowing version control, collaboration through pull requests, and seamless automated deployment of dashboards. Git Sync offers a scalable way to manage dashboards in complex environments, enhancing traceability, auditing, and consistency across multiple instances."
canonicalURL: "https://www.msbiro.net/posts/grafana-dashboard-configuration-with-gitops/"
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
    alt: "<alt text>"
    caption: "<text>"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---
Starting with version 12, Grafana introduces the ability to configure dashboards using a GitOps approach through an experimental feature called **Git Sync**.

This is a particularly interesting capability that can help manage dashboards in large and complex environments.

Git Sync is available as an experimental feature in both Grafana OSS and Enterprise editions. Activation can also be requested for the Cloud version (currently available as a private preview).

You can find the relevant documentation [in this page](https://grafana.com/docs/grafana/latest/observability-as-code/provision-resources/intro-git-sync/), and below I am including a demo video.

{{< youtube 6RTNUnPUFS4 >}}
