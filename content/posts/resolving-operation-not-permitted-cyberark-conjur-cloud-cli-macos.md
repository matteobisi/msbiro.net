---
title: "Resolving 'Operation Not Permitted' for CyberArk Conjur Cloud CLI on macOS"
date: 2025-01-17T16:20:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["conjur cloud","conjur cli","macos"]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#description: "Desc Text."
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
As a consultant, it's always a pleasure to explore new tools, and since the end of 2024, we have been experimenting with CyberArk's SaaS offering.

The first component we started working with is Conjur Cloud, the SaaS version of Conjur Enterprise, which we are already very familiar with.
Conjur Cloud features an impressive UI that allows users to configure and manage most settings seamlessly.

Like Conjur Enterprise, it also has its own dedicated CLI, available for download on the CyberArk Marketplace. After installing the Conjur Cloud CLI on macOS 15.2, I encountered the following error when attempting to execute it:

```
conjur -version
zsh: operation not permitted: ./conjur
```

After some troubleshooting, I discovered that the binary had been quarantined by macOS 15.2. Running the following command confirmed this:

```
xattr -l /Applications/ConjurCloudCLI.app/Contents/Resources/conjur/conjur
```

where I got the following output:

```
com.apple.quarantine: 0187;678a416a;Microsoft\\x20Teams\\x20WebView;
```

To resolve this issue, I removed the quarantine attribute using the following command:

```
xattr -d com.apple.quarantine /Applications/ConjurCloudCLI.app/Contents/Resources/conjur/conjur
```

After applying this fix, I was able to successfully launch the CLI:

```
conjur --version
Conjur Cloud CLI version 1.1.2
```
