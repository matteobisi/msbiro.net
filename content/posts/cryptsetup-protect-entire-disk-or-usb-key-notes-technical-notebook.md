---
title: "cryptsetup: How to Protect Entire Disks or USB Keys – Notes on technical_notebook"
date: 2024-07-15T18:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["cryptsetup","encryption","oss","technical_notebook"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#description: "Desc Text."
canonicalURL: "https://www.msbiro.net/posts/cryptsetup-protect-entire-disk-or-usb-key-notes-technical-notebook/"
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
I have been using an encrypted USB drive for several years, which unlocks with biometric access.  
Recently, I started wondering how to achieve the same level of protection with other disks or USB keys.

The answer is cryptsetup, a utility included in most Linux distributions.  
I’ve done some tests and documented how to use it in a [repository I’ve named technical_notebook](https://github.com/matteobisi/technical_notebook).

I’ll use technical_notebook as a personal notebook—it will contain commands, concepts, and useful links.  
The purpose of the repo is to help me remember these details, keep them easily accessible, and perhaps assist others who have similar needs.