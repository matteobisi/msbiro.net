---
title: "The Silent Heist: How Distillation Attacks Are Reshaping the Global AI Landscape"
date: 2026-02-23
tags: ["ai", "cybersecurity", "anthropic", "distillation", "geopolitics"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "An analysis of the recent distillation attacks against Anthropic's Claude models by three major Chinese AI companies, explaining what distillation is and its implications for the global AI landscape."
canonicalURL: "https://www.msbiro.net/posts/distillation-attacks-anthropic-vs-chinese-ai/"
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
    alt: "AI abstract network representing distillation"
    caption: "AI model distillation"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction to the AI Frontier

The AI race has largely boiled down to a high-stakes contest between the US and China. On one side, established US companies like Anthropic, OpenAI, Google, and X have continuously pushed the boundaries of frontier AI models. Anthropic, the research lab behind Claude, is best known for its focus on AI safety and its unique 'constitutional' approach to alignment.

Meanwhile, several Chinese tech firms have been fast-tracking models to compete with the best systems coming out of the US. This competition reached a turning point when Anthropic revealed it had been targeted by industrial-scale 'distillation attacks' from three major Chinese AI labs.

## What is a Distillation Attack?

In machine learning, 'distillation' is usually a standard, legitimate technique. It works by using the outputs of a massive 'teacher' model to train a smaller 'student' model, giving you similar performance with much less computing power.

However, a distillation *attack* occurs when this process is performed illicitly against a competitor's proprietary model. The attacker programmatically queries a target model, sometimes millions of times, to harvest high-quality training data. By doing so, they can acquire advanced AI capabilities at a fraction of the time and financial cost it took the original developer to train the frontier model from scratch. Anthropic noted that these attacks involved using commercial proxies to bypass regional restrictions and setting up "hydra clusters" of fraudulent accounts to scale their efforts undetected.

## The Three Attacks Unveiled by Anthropic

According to Anthropic's report, three major Chinese AI companies engaged in significant distillation campaigns to extract capabilities from the Claude models. The reported attacks are as follows:

| Attacker | Exchanges | Description |
| :--- | :--- | :--- |
| **DeepSeek** | > 150,000 | Targeted reasoning capabilities, using Claude as a reward model. Generated synchronized traffic, harvested 'chain-of-thought' data, and looked for ways to navigate sensitive topics. |
| **Moonshot AI** | > 3.4 million | Focused on agentic reasoning, tool use, computer vision, and coding. Employed hundreds of fraudulent accounts and attempted to reconstruct underlying reasoning traces. |
| **MiniMax** | > 13 million | Targeted agentic coding, orchestration, and tool use capabilities. Rapidly pivoted strategy to capture capabilities from new models within 24 hours of release. |

*Note: For extended information and technical details on these attacks, please refer to the source article in the References section.*

## Outro and Implications

While Europe continues to focus on regulatory frameworks like the AI Act and hopes to cultivate domestic AI champions capable of competing with State-of-the-Art models, the fundamental market reality remains a contest between the US and China. 

The Anthropic report adds a significant new layer to this geopolitical and technological rivalry. It'll be fascinating to see if these companies respond to such a public call-out. While the tech world is used to fights over web scraping, using distillation to effectively clone a competitor’s multi-million-dollar model is a much more aggressive move. What's more, many of these Chinese models are released as open-source. If these models were trained using capabilities illicitly extracted from commercial systems, it significantly amplifies the potential damage, as those unchecked capabilities are then freely proliferated globally.

We'll also need to keep a close eye on how US regulators respond. Given Anthropic's concerns that these actions undermine US export controls and pose national security risks, there may be increasing pressure to create a unified, stringent front to secure frontier AI models against such extraction behaviors in the future.

### References
*   [Detecting and preventing distillation attacks | Anthropic](https://www.anthropic.com/news/detecting-and-preventing-distillation-attacks)
