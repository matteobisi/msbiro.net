---
title: "cryptsetup: How to Protect Entire Disks or USB Keys – Notes on technical_notebook"
date: 2024-07-15T18:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "cryptsetup", "encryption", "oss", "technical_notebook",
  "disk-encryption", "luks", "usb-security", "linux", "data-protection"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A hands-on guide using cryptsetup to encrypt entire disks and USB keys on Linux, based on real tests and examples from the technical_notebook repository. Learn essential commands, concepts, and practical tips to secure your portable drives with open-source tools and biometric access."
canonicalURL: "https://www.msbiro.net/posts/cryptsetup-protect-entire-disk-or-usb-key-notes-technical-notebook/"
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
I have been using an encrypted USB drive for several years, which unlocks with biometric access.  
Recently, I started wondering how to achieve the same level of protection with other disks or USB keys.

The answer is cryptsetup, a utility included in most Linux distributions.  
I’ve done some tests and documented how to use it in a [repository I’ve named technical_notebook](https://github.com/matteobisi/technical_notebook).

I’ll use technical_notebook as a personal notebook—it will contain commands, concepts, and useful links.  
The purpose of the repo is to help me remember these details, keep them easily accessible, and perhaps assist others who have similar needs.