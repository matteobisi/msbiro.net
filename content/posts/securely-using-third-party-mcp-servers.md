---
title: "Securely Working with Third-Party MCP Servers"
date: 2025-11-17
tags: [
  "cybersecurity", "open-source", "devsecops", "mcp", "owasp"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A guide to understanding and securely implementing third-party Model Context Protocol (MCP) servers, based on the OWASP GenAI security cheatsheet."
canonicalURL: "https://www.msbiro.net/posts/securely-using-third-party-mcp-servers/"
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
    alt: "Securely Working with Third-Party MCP Servers"
    caption: "Securely Working with Third-Party MCP Servers"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In the rapidly evolving landscape of AI and large language models (LLMs), the ability to connect these models to external tools and data sources is crucial for building powerful, automated applications. The **Model Context Protocol (MCP)** has emerged as a standard for this purpose, but its use also introduces new security challenges. This article explores how to work securely with third-party MCP servers, drawing insights from the recently released **OWASP GenAI security cheatsheet**.

## What are MCP Servers?

MCP provides a standardized way for LLM and agent hosts to connect with external tools, data, and prompt templates. It operates on a simple client-server model, enabling AI applications to perform actions beyond simple text generation, such as accessing databases, calling APIs, or interacting with internal systems.

The core components of MCP are:

- **MCP Host**: The application that manages clients and routes requests.
- **MCP Client**: The connector that communicates with a single MCP server.
- **MCP Server**: A program that exposes tools, resources, and prompts to clients.

By using MCP, developers can unlock significant automation capabilities, but this also creates a new attack surface that, if not properly secured, can lead to data theft, malicious code execution, and system sabotage.

## The Role of OWASP in Cybersecurity

The **Open Web Application Security Project (OWASP)** is a non-profit foundation dedicated to improving software security. It is a global community of developers, security researchers, and corporate members who produce free and open-source articles, methodologies, documentation, tools, and technologies.

For anyone working in cybersecurity, OWASP is an indispensable resource. Their publications, such as the famous **OWASP Top 10**, provide authoritative and up-to-date guidance on the most critical security risks. Following OWASP's work is essential for staying informed about emerging threats and best practices in application security.

## Key Takeaways from the OWASP MCP Cheatsheet

The OWASP GenAI security project recently published a cheatsheet titled **"A Practical Guide for Securely Using Third-Party MCP Servers."** This guide is intended for developers and organizations that plan to use third-party MCPs. Here are some of the key takeaways:

### Understand the Vulnerability Landscape

The cheatsheet highlights several common attack patterns targeting MCP implementations:

- **Tool Poisoning & Rug Pull Attacks**: Malicious commands are embedded within a tool's description or parameters. A "rug pull" occurs when a legitimate tool is replaced with a malicious one.
- **Prompt Injection**: Attackers craft malicious inputs to hijack the model's context and force it to perform unintended actions.
- **Memory Poisoning**: An agent's memory is corrupted with false information, leading to flawed decision-making or privilege escalation.
- **Tool Interference**: Using multiple MCP servers can lead to unintended tool execution chains, causing data leaks or denial-of-service loops.

### Implement Robust Client Security

The MCP client is the first line of defense against malicious servers. The cheatsheet recommends several security measures:

- **Trust Minimization**: Always validate manifests, enforce schemas, and use allowlists.
- **Sandbox Execution**: Run clients in restricted environments (e.g., Docker containers) to limit the impact of a compromised server.
- **Just-in-Time (JIT) Access**: Grant tools temporary, narrowly scoped permissions.
- **UI Transparency**: Expose full tool descriptions and permissions to users before execution.

### Secure Server Discovery and Verification

The process of identifying, verifying, and connecting to MCP servers must be secure:

- **Verify Origin Before Connecting**: Only connect to servers from a trusted registry.
- **Pin Versions**: Maintain a manifest of approved server and tool versions and use checksums to verify their integrity.
- **Staged Rollout**: Test new servers in a staging environment before promoting them to production.

### Establish Strong Governance

A formal governance strategy is essential for ensuring that only vetted and monitored MCP servers are used. This includes:

- **A Trusted MCP Registry**: A central registry for all approved servers.
- **A Formal Governance Workflow**: A process for submitting, scanning, reviewing, and deploying new servers.
- **Clear Roles and Responsibilities**: Defined roles for developers, security reviewers, and operators.

## Useful Links

- **OWASP Website**: [https://owasp.org/](https://owasp.org/)
- **OWASP GenAI Project**: [https://genai.owasp.org/](https://genai.owasp.org/)
- **OWASP MCP Cheatsheet**: [https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/](https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/)

By following the guidance from OWASP and implementing the security controls outlined in the cheatsheet, organizations can safely leverage the power of MCP while protecting themselves from emerging AI-related threats.
