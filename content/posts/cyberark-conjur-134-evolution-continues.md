---
title: "CyberArk Conjur 13.4 – The Evolution Continues"
date: 2024-10-09T07:30:03+00:00
tags: [
  "cyberark", "conjur", "13.4",
  "secrets-management", "release", "automation", "dynamic-configuration", "external-secrets-operator", "enterprise-security"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "CyberArk Conjur 13.4 introduces exciting new features including syncing empty safes from Vault for improved policy automation, dynamic application configuration through the conjur.yml file, and extended support for regex queries in the External Secrets Operator. This release marks another step in the continuous enhancement of Conjur Enterprise, making it more powerful and flexible for enterprise secrets management."
canonicalURL: "https://www.msbiro.net/posts/cyberark-conjur-134-evolution-continues/"
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
About a month ago, a new release of Conjur Enterprise was launched—now at version 13.4, bringing exciting new features to the product!

Here are my top three favorites, though there are many more updates, which you can find [here](https://docs.cyberark.com/conjur-enterprise/13.4/en/content/enterprise/whatsnew.htm?TocPath=Get%20started%7C_____2):

- Sync of empty safes from Vault: This is essential for managing policy creation through automation.
- Dynamic application configuration: It is now possible to modify various Conjur configuration parameters that previously had to be set when creating the container. Now, they are all included in the usual conjur.yml.
- Extended ESO support: The External Secrets Operator can now use regex in findByName and findByTags.

As has been the case for several releases, I’d like to reiterate that CyberArk’s development of Conjur is moving quickly, and with each release, the product becomes more and more complete.

**Well done, CyberArk!**
