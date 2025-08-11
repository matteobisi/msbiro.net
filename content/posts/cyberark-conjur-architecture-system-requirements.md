---
title: "CyberArk Conjur: A Quick Overview of Architecture and System Requirements"
date: 2022-07-24T11:40:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "conjur", "cyberark", "secrets manager",
  "enterprise-security", "architecture", "system-requirements",
  "high-availability", "kubernetes", "docker", "podman", "container-runtime"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post provides a comprehensive overview of CyberArk Conjur Enterprise architecture, detailing its multi-node cluster design with auto-failover capabilities, follower deployment for scaling, and essential system requirements for production and test environments. Essential reading for anyone planning to deploy Conjur as an enterprise-grade secrets manager."
canonicalURL: "https://www.msbiro.net/posts/cyberark-conjur-architecture-system-requirements/"
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
As I wrote in [my last post](/posts/why-you-need-kubernetes-secrets-manager/), CyberArk Conjur is an enterprise secrets manager.
, CyberArk Conjur is an enterprise secrets manager.  

In this post, I’ll provide an architecture overview along with the main system requirements.  
Conjur is currently available in two versions: Enterprise and open source (known as OSS).   
A "cloud" version will be available soon, offered as a SaaS solution.  

This post focuses on the Enterprise version, which is similar but not identical to the OSS version.  

## Architecture

Conjur is distributed as a Docker image, making deployment and configuration fast and secure.  

The main services included in the appliance image are:  

   - nginx: Provides access to the administrative GUI and the API set for interacting with secrets.
   - PostgreSQL database: Two databases, one for configurations and secrets, and one for audit logs.
   - Conjur appliance: The core product.
   - syslog-ng: Used to collect audit and access logs.  

The smallest common configuration for Conjur Enterprise is a three-node cluster with auto-failover, as shown in the following diagram:  

![basic Cluster Diagram](conjur-basic-arc.png)

The cluster architecture follows an active-passive model, with high availability guaranteed by a load balancer placed in front of the cluster. This load balancer switches the node accessed by applications in case of a failure.  

Within the cluster, one node acts as the "leader node," handling all read/write operations and providing the API for applications to interact with secrets.  

The secondary nodes are called "standbys" and can be either synchronous or asynchronous, depending on the PostgreSQL replication setup.   
These nodes are inactive and do not provide API services.  
  
If the leader node fails, after a configurable period, the standbys will elect a new leader using etcd and the [Raft consensus algorithm](https://docs.cyberark.com/conjur-enterprise/latest/en/content/deployment/highavailability/deploy-auto-failover-intro.htm).

If auto-failover is not configured, manual promotion is required.  

It is also possible to configure a primary site with auto-failover and a secondary site with asynchronous standbys (without auto-failover),  ready to be promoted manually.

The final component of the architecture is the "follower," a read-only replica of the leader. Followers allow secrets to be read at scale and are horizontally scalable components typically configured behind a load balancer to handle all types of read requests, including authentication, permission checks, and secret fetches.  

Followers are usually installed on a Kubernetes cluster to serve applications with low latency.  

The architecture diagram with followers could be expanded as shown below:  

![complete conjur Cluster Diagram](conjur-complete-arch.png)

## System Requirements

Production environment:  

   -  4 cores
   -  8 GB RAM
   - 50 GB hard drive  
  
Test environment:
   - 2 cores
   - 4 GB RAM
   - 20 GB hard drive

Supported container runtimes:
   - Docker 1.13 or later
   - Mirantis Container Runtime (MRC) 19.x, 20.10 on RHEL 8.x
   - Kubernetes and OpenShift (only for the follower)
   - Podman 3.3 and 3.4  

The host server should run an operating system supported by the container runtime (the most popular are RHEL Server, RHEL-based, or Ubuntu Server LTS). To read more detail or updated versions please [refer to this page](https://docs.cyberark.com/conjur-enterprise/latest/en/content/deployment/platforms/dap-sysreqs-server.htm?tocpath=Setup%7CConjur%20Enterprise%20requirements%7C_____1).  

If the customer environment also includes a CyberArk Vault server that needs to be synchronized with Conjur, the following are [the requirements](https://docs.cyberark.com/conjur-enterprise/latest/en/content/deployment/platforms/dap-sysreqs-synchronizer.htm?tocpath=Setup%7CConjur%20Enterprise%20requirements%7C_____2) for a CyberArk Vault Synchronizer server:  

   - 4 cores
   - 8 GB RAM
   - Windows Server 2012 R2 or later
  