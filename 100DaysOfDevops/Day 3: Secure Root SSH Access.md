# Day 03 - Disable Direct Root SSH Login on Linux Servers

## Overview

This challenge focuses on strengthening SSH security by disabling direct root login on all application servers. Allowing direct SSH access to the root account increases security risks, as attackers often target the root user during brute-force attacks.

The objective is to configure all application servers to prevent direct root SSH access while still allowing administrative access through privileged users and sudo.

---

## Objective

- Disable direct root SSH login on all application servers
- Update the SSH daemon configuration
- Restart the SSH service to apply changes
- Verify the configuration on each server
- Improve overall server security posture

---

## Services Used

| Service | Purpose |
|----------|----------|
| SSH | Remote server administration |
| OpenSSH Server (sshd) | Handles incoming SSH connections |
| systemctl | Manage Linux services |
| sudo | Execute administrative commands |
| grep | Verify configuration settings |

---

## Implementation

### Connect to the Target Server

Log in to the application server using a non-root user account.

```bash
ssh tony@stapp01
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote systems |
| `tony` | User account used for login |
| `stapp01` | Target application server |

> Repeat the same process for `stapp02` and `stapp03` using the appropriate users.

---

### Switch to the Root User

Obtain administrative privileges to modify the SSH configuration.

```bash
sudo su -
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `sudo` | Execute commands with elevated privileges |
| `su` | Switch user account |
| `-` | Load the target user's environment and profile |

---

### Modify the SSH Configuration

Open the SSH daemon configuration file.

```bash
vi /etc/ssh/sshd_config
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `vi` | Linux text editor |
| `/etc/ssh/sshd_config` | Main SSH daemon configuration file |

Locate the following line:

```text
#PermitRootLogin yes
```

Update it to:

```text
PermitRootLogin no
```

This setting disables direct SSH login for the root user.

---

### Restart the SSH Service

Apply the configuration changes by restarting the SSH daemon.

```bash
systemctl restart sshd
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `systemctl` | Manage system services |
| `restart` | Stop and start the service |
| `sshd` | OpenSSH daemon service |

---

## Verification

### Verify SSH Configuration

Confirm that direct root login has been disabled.

```bash
grep PermitRootLogin /etc/ssh/sshd_config
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `grep` | Search for matching text |
| `PermitRootLogin` | SSH configuration directive being verified |
| `/etc/ssh/sshd_config` | SSH daemon configuration file |

Expected Output:

```text
PermitRootLogin no
```

The important part is:

```text
PermitRootLogin no
```

which confirms that direct root SSH access has been disabled.

---

### Repeat Configuration on Remaining Servers

Perform the same steps on the remaining application servers:

| Server | User |
|----------|----------|
| stapp01 | tony |
| stapp02 | steve |
| stapp03 | banner |

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
│ to App Server       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Switch to Root      │
│ User                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Edit SSH Config     │
│ PermitRootLogin no  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Restart SSH Service │
│ sshd                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Settings     │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh user@server` | Connect to a remote server |
| `sudo su -` | Switch to the root user |
| `vi /etc/ssh/sshd_config` | Edit SSH configuration |
| `systemctl restart sshd` | Restart SSH service |
| `grep PermitRootLogin /etc/ssh/sshd_config` | Verify SSH configuration |

---

## Key Learnings

- Learned how to secure Linux servers by disabling direct root login.
- Understood the importance of SSH hardening.
- Modified SSH daemon configuration safely.
- Applied configuration changes using systemd services.
- Verified security settings using command-line tools.

---

## Best Practices

- Disable direct root SSH access on production servers.
- Use named user accounts with sudo privileges instead of root.
- Regularly review SSH configuration settings.
- Restrict SSH access using firewalls and allowlists where possible.
- Test SSH access before ending an administrative session.

---

## Challenge Status

✅ Successfully Completed
