---
title: "Docker Hardened Images Are Now Free and Open Source"
date: 2025-12-18T09:00:00+01:00
tags: [
  "docker", "security", "supply-chain", "hardened-images", "open-source", "mcp"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Docker has made a significant move by releasing their Hardened Images catalog as free and open source. This post explores what this means for developers, the inclusion of Helm charts and MCP servers, and how the enterprise model supports this initiative."
canonicalURL: "https://www.msbiro.net/posts/docker-hardened-images-free/"
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
    alt: "Docker Hardened Images"
    caption: "Docker Hardened Images now Open Source"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

I've already touched the hardened images theme in the past talking how this theme is important in today's world. Reducing the attack surface of our containers is not just a "nice to have" anymore; it is a fundamental requirement for a secure software supply chain. In an era where vulnerabilities can be exploited within hours of disclosure, starting with a secure base is half the battle.

{{< youtube 21FE7_9DsBM >}}

That is why the recent move by Docker is so significant.



## Democratizing Security

Docker has officially announced that their **Hardened Images (DHI)** catalog is now **free and open source** under the Apache 2.0 license. You can read the official announcement [here](https://www.docker.com/blog/docker-hardened-images-for-every-developer/).

Previously, access to high-quality, minimal, and battle-tested images often required paid subscriptions or significant internal effort to maintain. By opening this up, Docker is essentially democratizing access to secure software foundations.  
This aligns perfectly with the shift left philosophy, giving every developer, regardless of budget, the tools to build securely from day one.

### What's Included?

This isn't just a dump of a few container images. The release is surprisingly comprehensive:

1.  **The Images:** A catalog of minimal images with near-zero CVEs. These come with verifiable SBOMs (Software Bill of Materials) and SLSA Build Level 3 provenance.
2.  **Hardened Helm Charts:** To bridge the gap between a secure image and a secure deployment, they have also open-sourced Hardened Helm Charts. This ensures that the configuration *around* your container is as secure as the container itself.
3.  **MCP Servers:** This is a forward-looking addition. Docker is expanding security to the AI layer by offering hardened versions of **Model Context Protocol (MCP)** servers. If you are building agents or integrating LLMs, having a hardened interface for things like MongoDB or GitHub is a game-changer.

Anyone out there stil looking for bitnami chart replacement? 🤔

## The Business Model: Sustainability vs. Charity

It is natural to ask: "How does Docker make money if they are giving this away?"

This is a classic—and very smart—open-core/freemium strategy. While the *artifacts* (the images) are free, the *guarantees* are what you pay for. Docker has structured this to support the community while retaining a strong value proposition for enterprises.

You can compare the plans in detail [here](https://www.docker.com/products/hardened-images/#compare), but the breakdown is essentially this:

*   **Free (Open Source):** You get the hardened images, the SBOMs, and the transparency. You rely on community support and the standard release cycle.
*   **Enterprise:** This is where the SLAs kick in.
    *   **7-Day Fix SLA:** Docker commits to fixing critical CVEs within 7 days.
    *   **Compliance:** Access to FIPS-enabled and STIG-ready images, which are non-negotiable for government and highly regulated industries.
    *   **CIS Benchmarks:** Compliance with Center for Internet Security benchmarks.
    *   **Support:** Direct support channels rather than just community forums.

There is also an **Extended Lifecycle Support (ELS)** add-on, which provides up to 5 years of security coverage after upstream support ends; a lifesaver for legacy enterprise applications that cannot be refactored easily.

## Why This is a Good Move

This approach benefits everyone:

- for **Developers**, it removes friction. You don't need procurement approval to start using a secure Redis or Nginx image. You just pull it. This increases adoption, which in turn leads to more eyes on the code and a stronger ecosystem.
- for **Docker**, it cements their position as the standard-bearer for container security. By becoming the default choice for secure base images, they funnel successful startups and growing companies into their Enterprise funnel when those companies eventually need FIPS compliance or SLA guarantees.
- for **competitors**, what will their move be now? There are alternative vendors out there with strong solutions, will they react in some way?

If you haven't looked at your base images recently, now is the time to swap them out for something hardened.
