---
title: "CyberArk Conjur, authenticators and integrations"
date: 2022-08-22T10:26:03+00:00
# weight: 1
# aliases: ["/first"]
tags: [
  "conjur", "authenticators", "spiffe", "integrations",
  "cybersecurity", "secrets-management", "kubernetes", "cloud-native",
  "mtls", "identity", "devops", "open-source"
]
author: "Matteo Bisi"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "detailing the variety of authenticators such as host/user API key, OIDC, AWS IAM, Kubernetes with SPIFFE-compliant mutual TLS, and more. Learn how these authenticators enable secure secrets retrieval and integrations with popular DevOps tools and cloud platforms, enhancing security and flexibility for dynamic environments."
canonicalURL: "https://www.msbiro.net/posts/cyberark-conjur-authenticators-integrations/"
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
During the past few weeks, I have described what a secrets manager is and provided an overview of the architecture and system requirements of CyberArk Conjur.  

A secrets manager can't do its job if it can't communicate with those who need to request  secrets, and that's where Conjur's magic comes in!  

The "authenticators" are responsible for the authentication process in Conjur and are specialized to do this in the most secure way, depending on the service.  
Here is the list of authenticators currently supported:  

| Authenticator   | Description |
| --------------- | --------------- |
| [authn](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Operations/Services/default_authn.htm)    | Defines the Conjur default authenticator. Authentication for both users and hosts is based on an ID and API key.     |
| [authn-oidc](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/OIDC/OIDC.htm)     | Leverages the identity layer provided by OIDC to allow applications to authenticate with Conjur and retrieve secrets needed for connecting to services such as a database.   |
| [authn-iam](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Operations/Services/AWS_IAM_Authenticator.htm)    | Enables an AWS resource to use its AWS IAM role to authenticate with Conjur.   |
| [authn-azure](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Operations/Services/azure_authn.htm)    | Enables an Azure resource to authenticate with Conjur   |
| [authn-jwt](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Operations/Services/cjr-authn-jwt-lp.htm)    | Enables an application to authenticate to Conjur using a JWT from a JWT Provider.    |
| [authn-gcp](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Operations/Services/cjr-gcp-authn.htm#GCP%C2%A0Auth)    | Enables a Google Cloud Platform resource to authenticate with Conjur    |
| [authn-k8s](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Integrations/k8s-ocp/k8s-k8s-authn.htm)    | Authenticates hosts that are Kubernetes resources, such as a Kubernetes namespace, deployment, stateful set, and others. Authentication is certificate-based using a mutual TLS connection.   |
| [authn-ldap](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Integrations/ldap/configure-ldap-authn.htm)    | Authenticates users based on an LDAP directory.     |

By default, after the initial setup, authn is the only authenticator enabled, and it is responsible for Conjur access using a username or API key (randomly generated between 51 and 56 characters).

When integrations are needed, for example with a Kubernetes cluster, you need to activate and configure the authn-k8s authenticator.
This authenticator can establish a secure mTLS connection compliant with the [SPIFFE framework](https://spiffe.io/docs/latest/spiffe-about/overview/), as described in the following diagram ([click here](https://developer.cyberark.com/blog/kubernetes-authentication/) for more details).


![Conjur authn-k8s authentication flow](k8s_authentication_flow.png)

If you need integrations with cloud identities that support JWT authentication, such as Google Apigee, you will enable the authn-jwt authenticator, which will perform the integrations as shown below:

![Conjur jwt authentication flow](cjr-authn-jwt-flow.png)

These were just two examples among the many possible secure integrations enabled by the authenticators. The following is a more complete list:

   - Kubernetes (local or cloud)
   - API gateway
   - Jenkins
   - Ansible
   - Puppet
   - Terraform
   - Cloud services

If the scenario described sounds interesting for your needs, Conjur could definitely be the right choice. Contact your sales representative at CyberArk or your [preferred CyberArk business partner](https://www.reevo.it/it/cloudnative-devops#tech-partners).  
