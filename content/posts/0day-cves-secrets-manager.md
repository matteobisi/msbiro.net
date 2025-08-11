---
title: "Urgent: Zero-Day CVEs Found in Two Major Secrets Managers — Have You Updated Yet?"
date: 2025-08-11T12:39:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["cybersecurity", "zero-day", "CVE", "secrets manager", "CyberArk", "HashiCorp", "software update", "vulnerability management"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This article highlights recent zero-day vulnerabilities discovered in CyberArk and HashiCorp secrets managers, emphasizes the importance of timely software updates, and offers practical advice for staying proactive about security patches."
canonicalURL: "https://www.msbiro.net/posts/page"
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
Today, my manager forwarded me this [article](https://www.darkreading.com/cybersecurity-operations/critical-zero-day-bugs-cyberark-hashicorp-password-vaults?utm_campaign=16543270-Minimus%20Weekly&utm_medium=email&_hsenc=p2ANqtz-8cp_mhmbyhxW0NhltnmD5c81JFTQ0ai1xnrE-jZNzZ-yY4l2jBN-ijlJgFbORNLIzQqEyGvFaaYC5lrasY1hi_aaOg0mD0m-EIVg1NiBj8EeCaMNw&_hsmi=375244214&utm_content=375244214&utm_source=hs_email) about several zero-day CVEs discovered in CyberArk and HashiCorp products. After some time spent researching online, ***I confirmed that both brands have fixed these CVEs by releasing updated versions!!*** 

![cve resolved](cve-resolved.jpg)

I'm not surprised that these two big corporations acted quickly and fixed the vulnerabilities; both are well-known and reliable!   
This event gave me an excuse to write this article and respond to one of the most common questions I get from my customers whenever I share news about a new release of a secrets manager:

- Do I have to update my environment?
- What changes in this release make the upgrade worthwhile?

In my mind, the answer is always the same: *“Do you care about having your company’s main secrets repository protected in the best possible way?”*

As I have already written on this blog, **CVEs are an inevitable "feature"** of every software over time. Initially, there are none, but as time goes on, CVEs accumulate: zero-day, low, medium, or high severity. This is not an opinion; it’s a fact based on how software works.

I understand that it may not be possible to follow every vendor upgrade, but please, the next time your consultant or vendor notifies you about an available update, remember the importance of that software for your company. **A zero-day CVE could be critical for a secrets manager, while less important for software like GIMP.**

Here are some useful tips to stay proactive if your consultant or vendor isn’t as responsive as I am:

- Set a reminder in your calendar to check for new releases and review the release notes.
- Subscribe to one or more cybersecurity news sites such as [Cyber Security News](https://www.linkedin.com/company/cybersecurity-news/posts/), [Hacking Articles](https://www.linkedin.com/company/hackingarticles/posts/?feedView=all), and [Dark Reading](https://www.darkreading.com/).
- Check your preferred CVE database by searching the product name, for example, the [NIST database](https://nvd.nist.gov/vuln/search#/nvd/home?resultType=records).