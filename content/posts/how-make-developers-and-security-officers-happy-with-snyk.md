---
title: "How Is It Possible to Make Both Developers and Security Officers Happy? Try Snyk!"
date: 2023-01-13T16:23:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["snyk","sast","sca","supply-chain"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#description: "Desc Text."
canonicalURL: "https://www.msbiro.net/posts/how-make-developers-and-security-officers-happy-with-snyk/"
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
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Being able to work safely in cybersecurity requires knowledge, attention to detail, and a solid portfolio of reliable software.  
One of the tools I have learned about and used in recent months is Snyk.  

![snyk certification](snyk-pro.jpeg)

Calling Snyk a “tool” isn’t quite accurate—it’s a security platform that offers a suite of tools capable of operating on any codebase, including:

  - [Code (SAST)](https://snyk.io/product/snyk-code/)
  - [Open Source (SCA)](https://snyk.io/product/open-source-security-management/)
  - [Containers](https://snyk.io/product/container-vulnerability-management/)
  - [Infrastructure as Code](https://snyk.io/product/infrastructure-as-code-security/)
  - [Cloud](https://snyk.io/product/infrastructure-as-code-security/)

In recent years, the amount of code produced has grown exponentially. The availability of countless open-source libraries and containers has accelerated development, but how can we be sure that all these resources are secure?

How can developers be responsible for the security of their own code as well as the work of others? How can security officers manage this scenario without becoming a bottleneck to productivity?

Snyk helps by integrating its tools into IDEs, Git repositories, and CI/CD pipelines, providing fast analysis and suggesting solutions for detected issues.

For example, Snyk can be installed as a VSCode plugin or set up to scan Git repositories. If an issue is found, it can automatically open a pull request proposing a fix.

Snyk is also fully integrable into customer environments, supporting both access and security policies to ensure full compliance with customer needs. Customizable dashboards and reports are available, enabling security officers to quickly understand the security status of a project.

Another interesting feature is that Snyk has built an open-source vulnerability database that catalogs vulnerabilities and provides examples and tutorials for developers.

The best part is that testing Snyk is easy—a free (limited) plan is available!

If you’re interested in learning more about Snyk, please [read the blog article](https://blog.sighup.io/snyk-and-shift-left-approach/) published by my colleague [Luca Bandini](https://www.linkedin.com/in/lucabandini/) about our experience with Snyk,   
which we also used to check the code of Fury, the Kubernetes distribution developed by [SIGHUP](https://sighup.io/).