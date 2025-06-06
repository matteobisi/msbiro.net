---
title: "From Manual to GitOps: Simplifying Grafana Dashboard Configuration with Git Sync"
date: 2025-05-12T11:34:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["grafana","dashboard","gitops"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#description: "Desc Text."
canonicalURL: "https://www.msbiro.net/posts/grafana-dashboard-configuration-with-gitops/"
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
Starting with version 12, Grafana introduces the ability to configure dashboards using a GitOps approach through an experimental feature called **Git Sync**.

This is a particularly interesting capability that can help manage dashboards in large and complex environments.

Git Sync is available as an experimental feature in both Grafana OSS and Enterprise editions. Activation can also be requested for the Cloud version (currently available as a private preview).

You can find the relevant documentation [in this page](https://grafana.com/docs/grafana/latest/observability-as-code/provision-resources/intro-git-sync/), and below I am including a demo video.

{{< youtube 6RTNUnPUFS4 >}}
