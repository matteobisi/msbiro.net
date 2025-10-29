---
title: "CyberArk Conjur 13.1 Released"
date: 2023-12-07T08:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "cyberark", "conjur", "13.1",
  "secrets-management", "release", "resiliency", "security", "vault-synchronizer", "kubernetes"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "CyberArk has released Conjur 13.1, a point update focusing on under-the-hood improvements that enhance the resiliency of Conjur followers. Key changes include major upgrades to the underlying container base image, PostgreSQL, and etcd versions, as well as enhanced flexibility in vault synchronization and secret segregation. This release is recommended for all Conjur Enterprise users seeking improved performance and stability."
canonicalURL: "https://www.msbiro.net/posts/cyberark-conjur-131-released/"
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
A couple of weeks ago, CyberArk released a new and interesting version of Conjur: 13.1.  

This point release is really interesting because it brings important under-the-hood updates that aim to increase the resiliency of followers.  
If you want to read more about this release, please [check out the article](https://blog.sighup.io/cyberark-conjur-13-1-has-been-relased-with-interesting-updates-under-the-hood/) I wrote on the SIGHUP blog.