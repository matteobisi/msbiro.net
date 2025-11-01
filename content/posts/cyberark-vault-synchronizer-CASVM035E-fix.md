---
title: "CyberArk Vault Synchronizer – CASVM035E Vault Name Is Missing: How to Fix It"
date: 2022-09-30T07:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "cyberark", "synchronizer", "CASVM035E",
  "vault-synchronizer", "secrets-management", "troubleshooting",
  "error-fix", "conjur-integration", "windows-service", "cybersecurity"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post addresses the CyberArk Vault Synchronizer error 'CASVM035E Vault name is missing' encountered during upgrade from version 11.7 to 12.7. It guides users through the straightforward fix by updating the INTEGRATION_VAULT_NAME value in the VaultConjurSynchronizer.exe.config file, restoring secrets synchronization functionality on Windows."
canonicalURL: "https://www.msbiro.net/posts/cyberark-vault-synchronizer-CASVM035E-fix/"
disableHLJS: true # to disable highlightjs
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
As you may know, one of the key components of the CyberArk Conjur architecture is the [Synchronizer](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Conjur/cv_Synchronizer-lp.htm), which is required to receive secrets from the Vault.

Last week, I took charge of an abandoned Synchronizer version 11.7 that had not been working for some time and also needed to be upgraded to the latest 12.7 release.

After completing the upgrade (check [this link](https://docs.cyberark.com/conjur-enterprise/latest/en/content/conjur/cv_upgrade.htm?tocpath=Integrations%7CCyberArk%20Vault%20Synchronizer%7C_____11) for the steps), the Windows service failed to start, and the log contained the following error:

```
[5] [main] FATAL VaultConjurSynchronizer.Service.SynchronizerService - VCSS006F Failed to start CyberArk Vault-Conjur Synchronizer Service: CASVM035E Vault name is missing.
```

After searching online, we found references to the CASOS log error for the Vault, where the documentation suggests contacting CyberArk support.

Fortunately, in the case of the Synchronizer, we were able to resolve the issue easily by editing the following file:

```
C:\Program Files\CyberArk\Synchronizer\VaultConjurSynchronizer.exe.config
```

The value of the key INTEGRATION_VAULT_NAME was blank. After filling it in with the correct vault name (as specified in vault.ini), the service started successfully and secrets synchronization resumed as expected.

