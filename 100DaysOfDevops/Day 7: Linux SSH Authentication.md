# Day 07 - Configure Password-less SSH Authentication Across Application Servers

## Overview

This challenge focuses on configuring SSH key-based authentication to enable secure, password-less access from the Jump Host to all application servers within the Stratos Datacenter.

Password-less SSH authentication is a common DevOps practice used for automation, script execution, configuration management, and centralized administration without requiring interactive password prompts.

---

## Objective

- Generate an SSH key pair for the `thor` user on the Jump Host
- Configure password-less SSH authentication to all application servers
- Enable secure access using the respective sudo users
- Verify successful key-based authentication

---

## Services Used

| Service | Purpose |
|----------|----------|
| SSH | Secure remote access |
| SSH Key Authentication | Password-less login |
| ssh-keygen | Generate SSH key pairs |
| ssh-copy-id | Copy public keys to remote servers |
| Linux | User and access management |

---

## Implementation

### Generate an SSH Key Pair on the Jump Host

Log in as the `thor` user and generate a new RSA key pair.

```bash
ssh-keygen -t rsa
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh-keygen` | Generates SSH public/private key pairs |
| `-t rsa` | Specifies RSA as the key algorithm |

When prompted:

```text
Enter file in which to save the key:
```

Press:

```text
Enter
```

to accept the default location.

When prompted for a passphrase:

```text
Enter passphrase:
```

Press:

```text
Enter
```

twice to leave the passphrase empty.

Expected Files Created:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

### Copy the Public Key to App Server 1

Install the public key for user `tony` on App Server 1.

```bash
ssh-copy-id tony@stapp01
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh-copy-id` | Copies the public key to a remote server |
| `tony` | User account on App Server 1 |
| `stapp01` | Hostname of App Server 1 |

You will be prompted for the user's password one final time.

---

### Copy the Public Key to App Server 2

Install the public key for user `steve` on App Server 2.

```bash
ssh-copy-id steve@stapp02
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh-copy-id` | Copies the public key to a remote server |
| `steve` | User account on App Server 2 |
| `stapp02` | Hostname of App Server 2 |

---

### Copy the Public Key to App Server 3

Install the public key for user `banner` on App Server 3.

```bash
ssh-copy-id banner@stapp03
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh-copy-id` | Copies the public key to a remote server |
| `banner` | User account on App Server 3 |
| `stapp03` | Hostname of App Server 3 |

---

## Verification

### Verify Password-less Access to App Server 1

```bash
ssh tony@stapp01
```

Expected Result:

```text
Login successful without password prompt
```

Exit the server:

```bash
exit
```

---

### Verify Password-less Access to App Server 2

```bash
ssh steve@stapp02
```

Expected Result:

```text
Login successful without password prompt
```

Exit the server:

```bash
exit
```

---

### Verify Password-less Access to App Server 3

```bash
ssh banner@stapp03
```

Expected Result:

```text
Login successful without password prompt
```

Exit the server:

```bash
exit
```

---

## SSH Authentication Workflow

```text
┌─────────────────────┐
│      Jump Host      │
│     User: thor      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Generate RSA Keys   │
│ ssh-keygen -t rsa   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Public Key Created  │
│ id_rsa.pub          │
└──────────┬──────────┘
           │
           ▼
 ┌───────────────────┐
 │   ssh-copy-id     │
 └─────────┬─────────┘
           │
 ┌─────────┼─────────┐
 │         │         │
 ▼         ▼         ▼
stapp01  stapp02  stapp03
  tony    steve    banner
 │         │         │
 ▼         ▼         ▼
authorized_keys updated
 │         │         │
 └─────────┼─────────┘
           │
           ▼
 Password-less SSH Access
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh-keygen -t rsa` | Generate an RSA SSH key pair |
| `ssh-copy-id user@host` | Copy public key to remote server |
| `ssh user@host` | Connect to a remote server |
| `exit` | Exit the remote session |

---

## Key Learnings

- Learned how SSH key-based authentication works.
- Generated RSA public/private key pairs.
- Configured password-less SSH access across multiple servers.
- Understood the role of the `authorized_keys` file.
- Verified secure remote access without password prompts.

---

## Best Practices

- Use SSH key authentication instead of passwords whenever possible.
- Protect private keys with appropriate file permissions.
- Rotate SSH keys periodically.
- Remove unused keys from `authorized_keys`.
- Consider using stronger key algorithms such as ED25519 in production environments.

---

## Challenge Status

✅ Successfully Completed
