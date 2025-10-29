---
title: "CyberArk Conjur 13.2 Released: Another Step in the Right Direction"
date: 2024-02-01T12:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "cyberark", "conjur", "13.2",
  "secrets-management", "release", "resiliency", "csi-driver", "vault-synchronizer", "kubernetes", "openshift"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "CyberArk released Conjur 13.2 with important bug fixes, support for OpenShift 4.14, and new key features including high availability for the Vault Synchronizer and enhanced support for the Container Storage Interface (CSI) driver. This release improves disaster recovery strategies and optimizes secret injection into Kubernetes pods, representing another solid step in Conjur’s ongoing evolution."
canonicalURL: "https://www.msbiro.net/posts/cyberark-conjur-132-released/"
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
One week ago, CyberArk released another update for Conjur Enterprise, which has now reached version 13.2—definitely another step in the right direction!  

This release includes the usual bug fixes, expands Conjur’s integrations (for example, OpenShift 4.14 is now supported), and, most importantly, adds two exciting new features:

- CyberArk Vault Synchronizer high availability support
- Enhanced Conjur support for the Container Storage Interface (CSI) driver

Synchronizer high availability enhances Conjur’s disaster recovery (DR) strategy.  
It is now possible to set up a "passive" Synchronizer in a DR site that is aware of the status of the primary Synchronizer and can take over in case of a failure.

This feature allows for optimized resource usage and more effective DR strategies for customers.  

The enhanced support for the Container Storage Interface (CSI) driver enables Conjur to inject secrets directly into the correct pod, bypassing the need for a sidecar or init container.
This new feature helps customers optimize resource usage on their clusters while continuing to fetch secrets securely.  

CyberArk Conjur is constantly evolving with each release, and in my opinion,  
its progress over the past year has been terrific. **Big kudos to CyberArk!**

As a final reminder, CyberArk Conjur is a secrets manager available in three different versions:

- [Enterprise](https://www.cyberark.com/products/secrets-manager-enterprise/)
- [Open Source](https://www.conjur.org/)
- [Cloud](https://docs.cyberark.com/conjur-cloud/latest/en/Content/ConjurCloud/cl_ConjurCloudOverview.htm?tocpath=Get%20started%7C_____1)