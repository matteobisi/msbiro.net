---
title: "Resolving Podman Log Rotation Issues in CyberArk Conjur Container 12.9 Deployments"
date: 2023-05-24T17:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "conjur", "cyberark", "podman", "12.9",
  "container-runtime", "log-rotation", "security",
  "container-management", "troubleshooting", "devops"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post addresses a log rotation issue seen in CyberArk Conjur 12.9 container deployments running on Podman. Unlike Docker, Podman requires the container to be recreated with the AUDIT_WRITE capability added and a specific permission set on the Nginx log directory for proper log rotation. The resolution was developed collaboratively with CyberArk support and is now documented for future updates. Essential guidance for operators using Podman with Conjur containers."
canonicalURL: "https://www.msbiro.net/posts/resolving-podman-log-rotation-issue-conjur-enterprise-129/"
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
CyberArk Conjur is released as an appliance and distributed as container images to enable fast, error-free setup.

The supported container runtimes include:

- Docker 20.10 or later
- Mirantis Container Runtime 20.10
- Podman 3.x, 4.x

While working with multiple Conjur environments in our labs and at customer sites, we noticed that log rotation (for Conjur, Nginx, cluster, etc.) did not function correctly on Podman, although it worked as expected on Docker.

After some investigation with the excellent CyberArk support team, we identified the solution:

The Conjur container needs to be re-created with the AUDIT_WRITE capability added:
```
podman run \
...
--cap-add AUDIT_WRITE \
...
registry.tld/conjur-appliance:12.9.0
```
To minimize noise in the Nginx logs, it is also necessary to set the following permission inside every Conjur container:  
```
chmod 701 /opt/cyberark/dap/log/nginx
```

The CyberArk support team was, as always, extremely helpful in assisting us and collaborating to find this solution.
These issues are now documented in the CyberArk documentation and should be addressed in future updates.  

If you experience the same issue, I recommend contacting CyberArk support to confirm whether this solution is applicable to your environment.  