# Day 05 - Install and Permanently Disable SELinux

## Overview

This challenge focuses on deploying SELinux (Security-Enhanced Linux) and configuring it to remain disabled after the next system reboot. SELinux provides an additional layer of security through mandatory access control (MAC) policies, helping protect systems from unauthorized access and privilege escalation.

As part of a phased security implementation, the required SELinux packages must be installed while configuring the system to keep SELinux disabled until further policy adjustments are completed.

---

## Objective

- Install the required SELinux packages
- Configure SELinux to be permanently disabled
- Avoid manually rebooting the server
- Verify the configuration changes
- Ensure SELinux remains disabled after the next scheduled reboot

---

## Services Used

| Service | Purpose |
|----------|----------|
| SELinux | Linux security framework |
| YUM | Package management utility |
| SSH | Remote server administration |
| sudo | Execute administrative commands |
| Linux Configuration Files | Manage system security settings |

---

## Implementation

### Connect to App Server 1

Log in to the target server where SELinux needs to be configured.

```bash
ssh tony@stapp01
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote Linux systems |
| `tony` | User account used for login |
| `stapp01` | Target application server |

---

### Switch to the Root User

Obtain administrative privileges required for package installation and configuration changes.

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

### Check Current SELinux Status

Verify the current SELinux operating mode before making any changes.

```bash
sestatus
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `sestatus` | Displays the current SELinux status and mode |

Example Output:

```text
SELinux status:                 enabled
Current mode:                   enforcing
Mode from config file:          enforcing
```

---

### Install Required SELinux Packages

Install the necessary SELinux packages and management utilities.

```bash
yum install -y selinux-policy selinux-policy-targeted policycoreutils
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `yum install` | Installs packages from configured repositories |
| `-y` | Automatically answers yes to installation prompts |
| `selinux-policy` | Provides SELinux policy definitions |
| `selinux-policy-targeted` | Installs the targeted SELinux policy |
| `policycoreutils` | Provides SELinux management utilities |

---

### Modify the SELinux Configuration

Open the SELinux configuration file.

```bash
vi /etc/selinux/config
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `vi` | Linux text editor |
| `/etc/selinux/config` | Main SELinux configuration file |

Locate one of the following entries:

```text
SELINUX=enforcing
```

or

```text
SELINUX=permissive
```

Update it to:

```text
SELINUX=disabled
```

This configuration ensures SELinux remains disabled after the next system reboot.

---

### Save the Configuration

Save the file and exit the editor.

```text
:wq
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `:wq` | Save changes and quit the editor |

---

## Verification

### Verify SELinux Configuration

Confirm that the configuration file has been updated successfully.

```bash
cat /etc/selinux/config
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `cat` | Displays the contents of a file |
| `/etc/selinux/config` | SELinux configuration file |

Expected Output:

```text
# This file controls the state of SELinux on the system.

SELINUX=disabled
SELINUXTYPE=targeted
```

The important part is:

```text
SELINUX=disabled
```

which confirms that SELinux will remain disabled after the next reboot.

> **Note:** The current runtime SELinux status does not need to change immediately. The task requirement is to ensure that SELinux is disabled permanently after the next scheduled reboot.

---

## Workflow

```text
┌─────────────────────┐
│ System Administrator│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Connect to Server   │
│ stapp01             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Check Current       │
│ SELinux Status      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Install Required    │
│ SELinux Packages    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Update Config File  │
│ SELINUX=disabled    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Configuration│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Disabled After      │
│ Next Reboot         │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `ssh tony@stapp01` | Connect to App Server 1 |
| `sudo su -` | Switch to the root user |
| `sestatus` | Display SELinux status |
| `yum install -y selinux-policy selinux-policy-targeted policycoreutils` | Install SELinux packages |
| `vi /etc/selinux/config` | Edit SELinux configuration |
| `cat /etc/selinux/config` | Verify configuration settings |

---

## Key Learnings

- Learned how SELinux enhances Linux security through mandatory access controls.
- Installed essential SELinux packages and management utilities.
- Configured SELinux to remain disabled after reboot.
- Verified SELinux settings through configuration files.
- Understood the difference between runtime SELinux status and persistent configuration.

---

## Best Practices

- Use SELinux in enforcing mode whenever application compatibility allows.
- Test SELinux policies in permissive mode before enforcing them in production.
- Maintain backups of configuration files before modification.
- Verify configuration changes before scheduled maintenance windows.
- Document all security-related changes for audit and compliance purposes.

---

## Challenge Status

✅ Successfully Completed
