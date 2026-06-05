# Day 01 - Linux User Setup with Non-Interactive Shell

## Overview

This challenge focuses on creating a dedicated Linux service account with a non-interactive shell. Service accounts are commonly used by applications and automated processes that do not require direct user access.

The objective is to create a user named `james` and prevent interactive logins by assigning the `/sbin/nologin` shell.

---

## Objective

- Create a Linux user named `james`
- Assign the non-interactive shell `/sbin/nologin`
- Prevent direct login access
- Verify successful user creation

---

## Services Used

| Service | Purpose |
|----------|----------|
| Linux User Management | Create and manage user accounts |
| useradd | Create new users |
| grep | Search user records |
| getent | Query system user databases |
| /etc/passwd | Store user account information |

---

## Implementation

### Connect to the Target Server

Connect to the application server where the user account needs to be created.

```bash
ssh <app-server>
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote Linux systems |
| `<app-server>` | Hostname or IP address of the target server |

---

### Create the User with a Non-Interactive Shell

Create the user `james` and assign `/sbin/nologin` as the login shell.

```bash
sudo useradd james -s /sbin/nologin
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `sudo` | Execute the command with administrative privileges |
| `useradd` | Create a new Linux user account |
| `james` | Username being created |
| `-s` | Specify the login shell for the user |
| `/sbin/nologin` | Prevents interactive login access |

---

### Verify User Creation

Check the user's entry in the local user database.

```bash
grep james /etc/passwd
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `grep` | Search for matching text |
| `james` | User account being searched |
| `/etc/passwd` | Linux user account database |

Expected Output:

```text
james:x:1002:1002::/home/james:/sbin/nologin
```

The important section is:

```text
/sbin/nologin
```

which confirms the user cannot log in interactively.

---

## Verification

### Verify User Entry

```bash
grep james /etc/passwd
```

Expected Output:

```text
james:x:1002:1002::/home/james:/sbin/nologin
```

### Alternative Verification

```bash
getent passwd james
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `getent` | Query entries from system databases |
| `passwd` | Search the user account database |
| `james` | User account being verified |

Expected Output:

```text
james:x:1002:1002::/home/james:/sbin/nologin
```

---

## Workflow

```text
┌─────────────────────┐
│ System Administrator│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Connect via SSH     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create User         │
│ james               │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Assign Shell        │
│ /sbin/nologin       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify User         │
│ Configuration       │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh <app-server>` | Connect to the target server |
| `sudo useradd james -s /sbin/nologin` | Create a non-login user |
| `grep james /etc/passwd` | Verify user entry |
| `getent passwd james` | Alternative verification method |

---

## Key Learnings

- Learned how to create Linux user accounts.
- Understood the purpose of service accounts.
- Learned how non-interactive shells improve security.
- Verified user accounts using multiple methods.
- Gained experience working with Linux user databases.

---

## Best Practices

- Use non-login shells for service accounts.
- Follow the principle of least privilege.
- Regularly audit user accounts.
- Avoid interactive access for automated services.
- Verify account configuration after creation.

---

## Challenge Status

✅ Successfully Completed
