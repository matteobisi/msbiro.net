---
title: "SIGHUP Secure Containers: how do you choose the oci base image for your workload?"
date: 2023-04-13T12:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "ssc", "secure containers", "security", "supply-chain",
  "container security", "container base images", "devsecops",
  "vulnerability management", "container catalog", "linux containers", "slas"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post discusses how to choose the right OCI base image for your workloads, emphasizing the importance of security, vulnerability management, and timely updates. It showcases SIGHUP’s Secure Containers service, which offers a curated, proactively patched container catalog with support, SLAs, and automation benefits to help teams maintain secure, compliant container supply chains."
canonicalURL: "https://www.msbiro.net/posts/sighup-secure-container-how-choose-base-image-security/"
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
I believe it’s important to start with a premise:  
In this article, I’ll talk about a product/service built and offered by my current employer, [SIGHUP](https://sighup.io).  

![SIGHUP SSC logo](sighup-ssc.png)

No one from my company has asked me to publish this blog post here; these are my honest opinions about [Secure Containers](https://sighup.io/secure-containers/).

Secure Containers is a paid service built by SIGHUP that provides secure, hardened, and updated [container base images](https://opensource.com/article/21/8/container-image).  
Developers working with containers and images now enjoy several advantages compared to the past, such as standardization, automation, and faster release times.

One of the most underestimated aspects of working with containers is the need to start from base images that must be chosen carefully to avoid issues such as:  

- Bugs
- CVEs
- Outdated images
- Malicious code

It’s clear that having constantly updated base images with the fewest possible CVEs is crucial.  
Any problems in the base image will be replicated in your container, which could then be running in production environments.  

Keeping base images updated and secure is a significant responsibility, often requiring dedicated attention from someone in the company—taking them away from other tasks.  

This is where the Secure Containers service can help, offering the following advantages:

- Comprehensive container catalog
- Proactively patched against all known CVEs and vulnerabilities
- Prometheus-friendly images
- Notifications, support status, and planned obsolescence
- Support and clear SLAs

If you’re interested in Secure Containers, please [visit the dedicated site](https://www.sighup.io/secure-containers/) to find more information and FAQs.  
You’ll also have the opportunity to enable a free trial of the service.  

If you’d like to read more about the security of container base images,   
[check out this article](https://thenewstack.io/container-security-101-a-guide-to-safe-and-efficient-operations/) where I’ll explore the topic in greater depth.