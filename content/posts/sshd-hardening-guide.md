---
title: "Back to Basics: My Opinionated 2025 sshd_config Hardening"
date: 2025-12-03T16:03:07Z
tags: [
  "cybersecurity", "devops", "linux", "open-source", "tutorial", "ssh_config", "devsecops", "openssh"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Back-to-basics sshd_config hardening for 2025: opinionated settings to disable root login, enforce key auth, modern ciphers, and timeouts. Secure your Linux servers from the ground up—no Kubernetes required"
canonicalURL: "https://www.msbiro.net/posts/back-to-basics-sshd-hardening/"
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
    alt: "A shield protecting a server."
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

In today's fast-paced tech landscape, it's common to find incredibly talented engineers mastering complex orchestrators like Kubernetes or crafting intricate Infrastructure as Code solutions. We're living in an era of high-level abstractions, which is fantastic for productivity. However, this focus on the 'new and shiny' can sometimes lead us to overlook the foundational bedrock upon which everything is built.

![sshd hardening](ssh-hardening.jpeg)


It might seem a bit old-school to write a blog post about hardening SSH in 2025. Yet, these 'basic' skills are more critical than ever. In a world of ephemeral infrastructure and complex supply chains, securing the front door to our systems remains a non-negotiable first step.

That's why I've decided to occasionally mix in some foundational content like this among my usual posts on DevSecOps, compliance, and AI.  
Consider this a back-to-basics guide, a chance to give some love to one of the most critical services we all manage: the SSH daemon.  

Let's step away from the YAML for a moment and ensure our systems are secure from the ground up.

---

## Preparing for Configuration Changes

The main configuration file for the SSH daemon is `sshd_config`, typically located at `/etc/ssh/sshd_config`. The syntax is simple `KEY value`, and we will go through the most important ones. This guide uses settings that are generally compatible with RHEL 9/10 and Ubuntu 22/24 LTS.

Before making any changes, it's a golden rule to back up the original file:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
```

**Important Note:** When modifying SSH configuration, always keep an active session open. If you make a mistake and lock yourself out, you can use that session to revert the changes. Only after you have successfully tested the new configuration by logging in with a *new* session should you close the old one.

## Core Security Hardening

Let's dive into the essential parameters to create a robust and secure SSH configuration.

### Disable Root Login

This is non-negotiable. Allowing direct login as the `root` user is a massive security risk, as it gives an attacker a known, all-powerful username to target.

```sshd_config
PermitRootLogin no
```

Users should log in with their unprivileged accounts and elevate their privileges using `sudo` when necessary. This creates an audit trail and enforces accountability.

### Disable Password-Based Authentication

Passwords can be weak, stolen, or brute-forced. Public key authentication is significantly more secure. Ensure password-based authentication is turned off and rely solely on SSH keys.

```sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```

This configuration makes it so the only way to log in is by using a cryptographic key pair.

### Use Modern Cryptographic Algorithms

Older cryptographic algorithms can be vulnerable to attacks. We need to enforce the use of modern, strong ciphers, key exchange algorithms (Kex), and message authentication codes (MACs).

The configuration below is a secure starting point for modern systems. These lists are ordered by preference.

```sshd_config
# Key Exchange Algorithms
KexAlgorithms sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org,gss-curve25519-sha256-,diffie-hellman-group16-sha512,gss-group16-sha512-,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256

# Ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

# Message Authentication Codes
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
```

**Note:** The available algorithms depend on your OpenSSH version and the libraries it was compiled with. You can check supported algorithms with `ssh -Q kex`, `ssh -Q cipher`, and `ssh -Q mac`.

### Limit User Access

If you know exactly which users or groups need SSH access, explicitly allow them. This follows the principle of least privilege.

```sshd_config
# Allow only specific users
AllowUsers user1 user2

# Or, allow only users in a specific group
AllowGroup sshusers
```

This is much more secure than leaving access open to all user accounts on the system.

### Configure Sensible Timeouts and Limits

Protect your server from brute-force attacks and hanging sessions.

*   `LoginGraceTime`: Sets how long a user has to complete authentication before the server disconnects.
    ```sshd_config
    LoginGraceTime 30s
    ```
*   `MaxAuthTries`: Defines the maximum number of authentication attempts per connection.
    ```sshd_config
    MaxAuthTries 3
    ```
*   `ClientAliveInterval` and `ClientAliveCountMax`: These settings terminate idle sessions. The server will send a null packet every `ClientAliveInterval` seconds. If it doesn't get a response after `ClientAliveCountMax` attempts, it disconnects the user.
    ```sshd_config
    ClientAliveInterval 300
    ClientAliveCountMax 2
    ```

## Applying and Testing Your Configuration
After modifying `/etc/ssh/sshd_config`, you must restart the `sshd` service for the changes to apply.
```bash
# RHEL / Ubuntu
sudo systemctl restart sshd
```
With the service restarted, it's time to test it. **From a new terminal window**—do not use your existing one—try to log in.
```bash
ssh user@your_server_ip
```
If you can log in successfully, congratulations! Your SSH server is now significantly more secure. If not, use your other active session to check the logs (as described in the troubleshooting section below) and revert the changes if necessary.

## Advanced: SSHD Management in Enterprises

While the settings above are perfect for standalone servers, managing SSH in a large enterprise environment introduces a different set of challenges. Here, user accounts are rarely local and access patterns are much more complex.

### Integrating with Centralized User Directories (LDAP/AD)

In most corporate environments, users are managed in a central directory like LDAP or Active Directory. You can configure SSH to use these directories for authentication and authorization.

A common approach is to use `sssd` (System Security Services Daemon) on Linux to connect the system to the remote directory. When using LDAP, it's crucial to secure the connection itself.

**Best Practices for LDAP Integration:**

*   **Use LDAPS or STARTTLS:** Always encrypt the connection to the LDAP server to prevent credentials from being transmitted in cleartext.
*   **Restrict Access with Filters:** Don't just allow any user in the directory to log in. Use LDAP filters in your `sssd` or `nss_ldap` configuration to limit access to specific groups (e.g., `(memberOf=cn=ssh-users,ou=groups,dc=example,dc=com)`).
*   **Limit Server-Side Lookups:** Configure `sssd` to cache credentials to reduce the load on the LDAP server and improve login times.

### The Challenge of SSH Key Management

We've established that passwordless authentication using SSH keys is the way to go. However, in a large organization, managing thousands of public keys can quickly become a nightmare.

*   **Attribution:** How do you know who `ssh-rsa AAA...xyz john.doe@example.com` belongs to? The comment is optional and can be changed by the user. An `authorized_keys` file can become an unmanageable list of cryptic keys.
*   **Offboarding:** When an employee leaves the company, how do you ensure their SSH key is removed from every single server they had access to? A forgotten key is a permanent backdoor.

### Centralizing SSH Key and Session Management

To solve these problems, enterprises use centralized tools to manage SSH access.

*   **Configuration Management (Ansible, Puppet, Chef):**
    Tools like Ansible can be used to maintain a central, version-controlled repository of `authorized_keys` files. An Ansible playbook can be run regularly to distribute the correct keys to all servers, ensuring that only active employees have access.

    *Example (`ansible/authorized_keys/users/`):*
    ```
    # In your Ansible variables or inventory
    users:
      - name: alice
        ssh_key: "ssh-rsa AAAA... alice@company.com"
      - name: bob
        ssh_key: "ssh-rsa BBBB... bob@company.com"
    ```
    Your playbook would then iterate through this list and apply it to the servers.

*   **SSH Key Lifecycle Management (CyberArk SSH Manager / Venafi):**
    Dedicated tools like CyberArk SSH Manager (formerly Venafi SSH Protect) automate the entire lifecycle of SSH keys. They provide discovery of existing keys, enforce rotation policies, and grant access based on centralized rules, removing the burden from individual administrators.

*   **SSH Certificate Authority (HashiCorp Vault):**
    A more advanced approach is to use a tool like HashiCorp Vault as an SSH Certificate Authority (CA). Instead of managing individual public keys on servers, you configure `sshd` to trust a single CA certificate.

    ```sshd_config
    # In /etc/ssh/sshd_config
    TrustedUserCAKeys /etc/ssh/trusted-user-ca.pub
    ```
    Users then request short-lived SSH certificates from Vault, often authenticating with their existing identity (like LDAP or OIDC). This provides:
    *   **Just-in-Time (JIT) Access:** Credentials that expire automatically.
    *   **One-Time Password (OTP):** Vault can be configured to require an OTP for certificate issuance.
    *   **Centralized Auditing:** All certificate requests are logged in Vault.

*   **Privileged Access Management (PAM) Systems:**
    In many large enterprises, direct SSH (or RDP) access to servers is completely forbidden. Instead, all access is brokered through a Privileged Access Management (PAM) solution (e.g., CyberArk, Delinea, Teleport). Users log into the PAM system, which then establishes and records the session to the target server on their behalf. This provides full session recording, keystroke logging, and a complete audit trail for compliance and security investigations.

## Troubleshooting Common Issues

When things go wrong, logs are your best friend.

### Finding SSH Logs

On modern systemd-based distributions like **RHEL 9/10** and **Ubuntu 22/24 LTS**, the best way to check logs is via `journalctl`.

```bash
# Follow live logs for the sshd service
sudo journalctl -u sshd -f

# View all past logs for the sshd service
sudo journalctl -u sshd
```

On some older or differently configured systems, logs might still go to `/var/log/auth.log` (Debian/Ubuntu family) or `/var/log/secure` (RHEL family).

For more detailed logs, you can temporarily increase the log level in `sshd_config`.

```sshd_config
LogLevel VERBOSE
```
`INFO` is the default, while `DEBUG` provides the most detail but is very noisy and should not be used in production long-term.

### Common Errors and Fixes

1.  **`Permission denied (publickey)`**: This is the most common error.
    *   **Server-Side**: Ensure the user's public key is correctly added to `~/.ssh/authorized_keys` on the server.
    *   **Client-Side**: Check that the user is providing the correct private key to their SSH client (`ssh -i /path/to/private_key user@host`).
    *   **Permissions**: This is critical. `sshd` is very picky about file permissions. Run these on the server:
        ```bash
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/authorized_keys
        ```

2.  **Connection Timed Out**:
    *   Check firewall rules on the server (e.g., `firewalld` on RHEL, `ufw` on Ubuntu) and any network firewalls in between. Ensure TCP port 22 (or your custom SSH port) is open.

## Conclusion
Hardening your SSH server is a foundational step in securing any Linux system. Security is a process of continuous improvement/regularly review your configurations, stay informed about new threats, and master the fundamentals.  

As a follow-up to this article, I will explore in a future one how HashiCorp Vault can centralize SSH secret management, automate credential rotation, and implement fine-grained access policies.

Keep these configurations in place and remember: the strongest security chain is built on solid fundamentals!
