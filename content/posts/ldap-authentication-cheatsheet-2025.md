---
title: "LDAP: A Nostalgic Dive into Authentication and Why It's Still Kicking in 2025"
date: 2025-11-22T15:37:05Z
draft: false
tags: ["ldap", "authentication", "iam", "activedirectory", "devsecops", "2025"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "A trip down memory lane to the world of LDAP. This post is a cheatsheet for modern engineers on how to configure application authentication with LDAP and why this technology is still relevant today."
canonicalURL: "https://msbiro.net/posts/ldap-authentication-cheatsheet-2025/"
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
    alt: "<alt text>"
    caption: "<text>"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Even in the cloud-native era, where everything is an API call away, some technologies from the past refuse to fade away. Recently, I found myself helping my team of talented engineers configure HashiCorp Boundary for Microsoft Active Directory authentication.  
I was surprised to see that they were not familiar with the concepts of LDAP, a technology that was a cornerstone of my career for years. After spending countless hours configuring Domino, Sametime, WebSphere Portal, and Connections with LDAP, the process felt like riding a bike.

This experience made me realize that while modern authentication systems like an IDP or OIDC are the new standard, the good old LDAP is often still running behind the scenes. Every corporation has (at least) one (typically Active Directory), and it's often the source of truth where user identities are born before being magically synced to the likes of Microsoft Entra ID or Okta.

So, for all the young engineers out there, here is a nostalgic blog post and a practical cheatsheet to help you navigate the world of LDAP authentication.

---

## What is LDAP?

LDAP stands for **Lightweight Directory Access Protocol**. It's an open, vendor-neutral, industry-standard application protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network. Invented in the early 1990s by Tim Howes, Steve Kille, and Wengyik Yeong, LDAP was designed as a lightweight alternative to the X.500 Directory Access Protocol (DAP).

An LDAP directory is a tree-like structure that stores information about objects. These objects can be anything from users and groups to computers, printers, and even conference rooms. Each entry in the directory has a **Distinguished Name (DN)**, which uniquely identifies it and defines its position in the hierarchy.

A DN is composed of several components, such as:
- **CN**: Common Name
- **OU**: Organizational Unit
- **DC**: Domain Component

For example, the DN for a user named `Matteo Bisi` in the `engineering` department of the `sighup.io` domain might look like this:

```
cn=Matteo Bisi,ou=engineering,dc=sighup,dc=io
```

### LDAP Ports and Security

LDAP communication often occurs over port **389** for cleartext (unencrypted) connections. Naturally as best practice you need to use a secure communication, **LDAPS** (LDAP over SSL/TLS) uses port **636**. When connecting to an LDAPS server, your application will likely need to trust the LDAP server's certificate. This often involves importing the LDAP server's Certificate Authority (CA) certificate into your application's trust store. Failing to do so will result in connection errors due to untrusted certificates.

### Principal Implementations Over the Years

Over the years, several vendors have implemented LDAP, creating a rich ecosystem of directory services. Some of the most notable ones include:

- **Novell eDirectory**: One of the early pioneers in the directory services space.
- **Microsoft Active Directory**: The most widespread LDAP implementation, found in almost every corporate environment.
- **OpenLDAP**: The leading open-source LDAP implementation, known for its flexibility and performance.
- **IBM Tivoli Directory Server (TDS)**: A robust directory server often used in large enterprise environments.

---

## The LDAP Authentication Cheatsheet

When you need to configure an application to authenticate against an LDAP directory, you will typically need to provide the following information.

### BaseDN

The **Base Distinguished Name (BaseDN)** is the starting point for all searches in the LDAP directory. It tells your application where to start looking for users and groups. It's often the root of your domain, but it can be a specific OU for better performance and security.

```
dc=sighup,dc=io
```

Or a more specific one:

```
ou=users,dc=sighup,dc=io
```

### Binding to the LDAP Server

To interact with the LDAP server, your application needs to "bind" to it. There are two primary ways an application can bind:

-   **Anonymous Binding**: If the LDAP server allows it, your application can bind anonymously. This means no specific username or password is provided. Anonymous binding is generally used for read-only access to public directories or when the application performs its own authentication after retrieving user details.
-   **Bind User (Service Account)**: More commonly, applications use a dedicated "bind user" (often a service account) with a specific password. This user needs appropriate Read-Only Access Control (RBAC) permissions to search the directory for users and groups. This approach is necessary when anonymous binds are disallowed or when the application needs to perform operations beyond simple lookups, even if just for reading specific attributes that might not be publicly accessible. If your application also needs to write to the directory (e.g., change passwords), the bind user will require write permissions.

### User Object Definition

You need to tell your application what attributes to use to identify a user. In Active Directory, `sAMAccountName` is a common choice, which is the pre-Windows 2000 logon name. In other LDAP implementations, it might be `uid` or `cn`.

Here are some common attributes used for user identification:

- `sAMAccountName`: The user's logon name (e.g., `jdoe`).
- `cn`: The user's common name (e.g., `John Doe`).
- `uid`: The user's unique identifier.
- `mail`: The user's email address.

### Search Filter

The search filter is a string that tells the LDAP server how to find the user object. It combines the user object definition with a placeholder for the username entered by the user.

A typical search filter for Active Directory looks like this:

```
(&(objectClass=user)(sAMAccountName=%s))
```

This filter tells the LDAP server to look for an object that is a `user` and whose `sAMAccountName` matches the username provided (`%s`).

### Group Definition

To manage permissions, you will often need to check if a user is a member of a specific group. You can do this by searching for the group and checking if the user is listed as a member.

A common way to check for group membership is to use the `memberOf` attribute in the user's search filter.

```
(&(objectClass=user)(sAMAccountName=%s)(memberOf=cn=boundary-users,ou=groups,dc=sighup,dc=io))
```

This filter checks if the user is a member of the `boundary-users` group.

### Troubleshooting Common Issues

When things go wrong, LDAP error messages can be cryptic. Here are a couple of common error codes and tools to help you debug.

#### Common Error Codes

-   **LDAP error code 49: Invalid Credentials**
    This is one of the most frequent errors. It means the username or password (or both) you are using to bind to the LDAP server is incorrect. Double-check the `bindDN` and password. Sometimes, it can also be caused by a locked user account or an expired password.
    A common cause in Active Directory is using the wrong format for the user. For example, using `cn=mbisi,ou=users,dc=sighup,dc=io` when the application expects `mbisi@sighup.io` or `SIGHUP\mbisi`.

-   **LDAP error code 32: No Such Object**
    This error indicates that the BaseDN or another part of the Distinguished Name (DN) in your configuration does not exist on the server. Carefully check your BaseDN, user search paths, and group search paths for any typos or incorrect OUs.

#### Debugging Tools

-   **`ldapsearch` (Command-Line)**: A powerful command-line utility available on most Linux systems for querying an LDAP server. It's invaluable for testing connection details, search filters, and attributes.

    Here's an example of how to use `ldapsearch` to find a user:
    ```bash
    ldapsearch -x -H ldap://ldap.sighup.io -b "dc=sighup,dc=io" -D "cn=binder,dc=sighup,dc=io" -w "bind_password" "(sAMAccountName=mbisi)"
    ```
    - `-x`: Use simple authentication.
    - `-H`: The LDAP server URI.
    - `-b`: The BaseDN to start the search from.
    - `-D`: The Distinguished Name (DN) to bind with.
- `-w`: The password for the bind DN.
    - The last part is the search filter.

-   **Softerra LDAP Browser (Windows GUI)**: For those working on Windows, Softerra LDAP Browser is an excellent free GUI tool that allows you to browse the LDAP directory, run searches, and inspect attributes in a user-friendly interface. It's perfect for visualizing the directory structure and testing your connection parameters before plugging them into your application.

---

## Key Takeaways

LDAP has been a central actor in the IAM landscape for more than a decade. While it may lack some modern features like native MFA, its flexibility and widespread adoption make it a technology that is still relevant today.

Understanding the basics of LDAP is a valuable skill for any engineer. It will not only help you in those rare moments when you need to configure an application for LDAP authentication, but it will also give you a better understanding of the foundations upon which modern identity and access management systems are built. So, the next time you hear "LDAP," don't dismiss it as a relic of the past. It's a piece of history that is still very much alive and kicking.
