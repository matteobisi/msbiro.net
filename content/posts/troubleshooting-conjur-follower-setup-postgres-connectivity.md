---
title: "Troubleshooting CyberArk Conjur Follower Setup and Postgres Connectivity"
date: 2022-11-21T11:23:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "conjur", "follower", "postgres", "troubleshooting",
  "cyberark", "database-connection", "replication",
  "network-issues", "container-debugging", "ssl-certificates", "devops"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post covers troubleshooting a CyberArk Conjur follower setup issue where the follower pod could connect to the Conjur API leader but failed to connect to the Postgres database, causing replication to stall and system errors. The solution involved verifying Postgres connectivity using openssl s_client with TLS, revealing a network load balancer misconfiguration that was subsequently corrected. Learn how to use this simple openssl command for effective container and network diagnostics."
canonicalURL: "https://www.msbiro.net/posts/pagetroubleshooting-conjur-follower-setup-postgres-connectivity/"
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
During our work with a CyberArk Conjur environment, we encountered strange behavior during the Conjur follower setup.

The setup process on the follower would start, the seed was received, imported, and expanded, but after a few more steps, the process would hang and end with a generic "System Error."

After displaying the error message, the Conjur follower would restart.

We performed troubleshooting inside the Conjur Follower pod and verified that the follower could connect to the Conjur API leader successfully, but it was unable to connect to the Postgres database and complete the initial replication.

The correct way to verify Postgres connectivity from the follower to the leader is with the following command:  
```
echo "" | openssl s_client -starttls postgres -connect <lb_DNS>:5432 -showcerts
```
If the server certificate is returned, Postgres connectivity is working as expected.

In our case, we were unable to retrieve the certificate, which pointed us to an issue with the network load balancer. A colleague was able to fix the problem there.

Thanks to CyberArk support for providing us with the openssl command, which is easy to run from the container or any server. We had tried other verification methods, but openssl s_client is readily available on most containers and servers.  

For more information about openssl s_client and its options, check out this helpful [blog post](https://www.misterpki.com/openssl-s-client/).  