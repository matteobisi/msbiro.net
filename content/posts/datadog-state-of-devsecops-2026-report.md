---
title: "The Exploitability Gap: Insights from Datadog’s State of DevSecOps 2026"
date: 2026-03-06T09:00:00Z
tags: [
  "cloud-native", "kubernetes", "cybersecurity",
  "devsecops", "datadog", "containers", "supply-chain"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
description: "Exploring the critical findings of the Datadog State of DevSecOps 2026 report, focusing on exploitable vulnerabilities, unmaintained libraries, and CI/CD security risks."
canonicalURL: "https://www.msbiro.net/posts/datadog-state-of-devsecops-2026-report/"
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
    image: "/img/datadog-2026-report.svg"
    alt: "Abstract visualization of Cloud-Native Supply Chain and Zombie Vulnerabilities"
    caption: "The evolving landscape of DevSecOps in 2026"
    relative: false
    hidden: false
    hiddenInList: false
    hiddenInSingle: false
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

### Intro

We have all been there: a Slack notification triggers an alert for a "Critical" CVE, and the scramble to patch begins. But as our clusters grow, so does the noise. The most jarring security stories are often the ones happening silently inside our own production environments.

Datadog recently released its [State of DevSecOps 2026 report](https://www.datadoghq.com/state-of-devsecops/), and the numbers provide a sobering reality check for anyone managing cloud-native infrastructure. The report reveals that 87% of organizations are currently running at least one known exploitable vulnerability in their deployed services. Even more concerning is that many of these services rely on libraries that have been abandoned by their maintainers. This is not just a theoretical problem; it is based on telemetry from thousands of real-world cloud environments, making the findings impossible to dismiss. 

As we look at these statistics, we have to stop asking why vulnerabilities exist and start questioning why our current processes continue to allow such high levels of exploitable risk to reach production.

### Core Concepts from the Report

The 2026 report highlights a significant gap between traditional vulnerability scanning and actual runtime risk. These are the key data points that define the current landscape of cloud-native security:

*   **Exploitability vs. Severity:** While 87% of organizations have exploitable vulnerabilities, only 18% of "critical" vulnerabilities remain truly critical once runtime context (like internet exposure) is considered.
*   **The EOL Risk Factor:** 10% of services run on end-of-life (EOL) language versions. These services are 35% more likely to harbor exploitable vulnerabilities compared to those on supported versions.
*   **The Speed Trap:** 50% of organizations adopt new library versions within 24 hours of release, significantly increasing exposure to "day-of-release" supply chain attacks.
*   **CI/CD Blind Spots:** Only 4% of organizations pin marketplace GitHub Actions to a full-length commit SHA, while 71% leave them completely unpinned, creating a massive supply-chain attack surface.

### Conclusions and My Personal Takeaways

The primary takeaway for DevSecOps and platform teams is that severity is no longer the metric that matters; exploitability and context are the new gold standards for survival. We must move away from the impossible goal of fixing every CVE and instead focus exclusively on the exploitable vulnerabilities that have a real path to impact in production. This shift marks our transition from "vulnerability management" to "exposure management."

From my perspective as a DevSecOps team lead, we need to stop fighting fires and start building better foundations. To navigate the landscape shown in the Datadog report, I advocate for three non-negotiable practices:

*   **Hardened Image Catalogs:** In 2026, using unverified base images is no longer acceptable. Companies must maintain a centralized, hardened catalog of base images that are pre-vetted and automatically rebuilt. This is the most effective way to eliminate the "Zombie" vulnerabilities that haunt those 10% of EOL services.
*   **OpenVEX for Prioritization:** We cannot patch everything. By using OpenVEX (Vulnerability Exploitability eXchange) reports, we can programmatically filter out vulnerabilities that are present in the code but not actually exploitable in our specific environment. This turns a list of thousands of alerts into a focused task list of the 18% that actually matter.
*   **Pipeline Pinning and Cooldowns:** We must enforce strict pinning for all external GitHub Actions to specific commit SHAs and implement a mandatory one-week "cooldown" period for new dependency versions. This directly mitigates the "day-of-release" supply chain risks highlighted in the report.

This week, I challenge you to look at your most critical CI/CD pipeline and check two things: Are your external actions pinned to specific commit hashes, and how is your team actually prioritizing the triage of exploitable vulnerabilities? 

It is a small, concrete step that moves your team away from a "hope as a strategy" model and toward a more resilient architecture. Let us stop counting vulnerabilities and start measuring our actual exposure.
