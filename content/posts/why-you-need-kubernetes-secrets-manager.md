---
title: "CyberArk Conjur - why you (probably) need an enterprise secrets manager"
date: 2022-07-19T16:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "conjur", "introduction", "secrets manager",
  "enterprise-security", "cloud-native", "secrets-management",
  "cybersecurity", "devsecops", "password-management"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Secrets management is critical to securing modern infrastructures, guarding sensitive information such as passwords, certificates, and keys from exposure and misuse. This post introduces CyberArk Conjur, an enterprise-grade secrets manager that protects secrets using programmable REST APIs, centralized security policies, and integration with CyberArk’s broader ecosystem. Learn why avoiding common mistakes like hardcoding secrets or pushing them to public repositories is vital, and how Conjur offers scalable, secure, and automated secrets management for cloud-native environments."
canonicalURL: "https://www.msbiro.net/why-you-need-kubernetes-secrets-manager"
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
Security is always a complex topic to address, as an error or omission in processes can lead to serious economic or reputational damage for a company.  
When we talk about "secrets," consider the following examples:  

   - Usernames
   - Database passwords
   - SSL certificates and keys
   - SSH keys
   - Cloud credentials  

Simply reading through this list helps to explain why this topic needs to be considered and handled carefully.  
Some common bad practices or risks include:  

   - Hardcoding secrets in code
   - Data breaches
   - Password leaks
   - Secrets pushed to public repositories  

With practices like lateral movement, just one compromised secret can be enough to compromise an entire environment. 
To help prevent these risks, there are tools known as "enterprise secrets managers." I’d like to start a series of posts on this blog about [CyberArk Conjur](https://docs.cyberark.com/conjur-enterprise/latest/en/content/deployment/highavailability/high_availability_overview.html?tocpath=Setup%7C_____1).  
Conjur allows you to avoid direct use of secrets by leveraging a set of REST APIs, making it a programmable tool that can be accessed via URL or open source utilities.  
Security is enforced through security policies without slowing down the developers involved.  
Corporate security can be further improved with the use of rotators, which programmatically change secret values.  
If other CyberArk software like PAS Vault is already in use, Conjur can be integrated using the Synchroniser component,  
providing the same level of security for cloud-native infrastructure.  

Conjur is available in two versions: [enterprise](https://docs.cyberark.com/conjur-enterprise/latest/en/content/resources/_topnav/cc_home.htm) and [open source](https://docs.conjur.org/Latest/en/Content/Resources/_TopNav/cc_Home.htm), each with [distinct features](https://www.cyberark.com/resources/solution-briefs/cyberark-conjur-enterprise-and-cyberark-conjur-open-source).   
  
In upcoming posts, I will explain details about the architecture, secrets management, and product news related to CyberArk Conjur.  