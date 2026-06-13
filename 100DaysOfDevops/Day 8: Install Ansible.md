# Day 08 - Install Ansible 4.9.0 Using pip3 on Jump Host

## Overview

This challenge focuses on setting up **Ansible** on the Jump Host to prepare for infrastructure automation and configuration management across the Stratos Datacenter.

As part of the initial Ansible testing phase, the requirement is to install **Ansible version 4.9.0** using **pip3** and ensure that the Ansible executable is globally accessible so that all users on the system can execute Ansible commands without specifying a full path.

---

## Objective

- Install Ansible version `4.9.0` using `pip3`
- Verify Python3 and pip3 availability
- Ensure the Ansible binary is globally accessible
- Validate the installation and version
- Confirm Ansible can be executed by all users

---

## Services Used

| Service | Purpose |
|----------|----------|
| Ansible | Configuration management and automation |
| Python 3 | Runtime environment for Ansible |
| pip3 | Python package manager |
| Linux | System administration |
| Environment Variables | Global command accessibility |

---

## Implementation

### Verify Python3 and pip3 Availability

Before installing Ansible, verify that Python3 and pip3 are available on the Jump Host.

```bash
python3 --version
```

```bash
pip3 --version
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `python3 --version` | Displays installed Python version |
| `pip3 --version` | Displays installed pip3 version |

Example Output:

```text
Python 3.x.x
pip x.x.x
```

---

### Install Ansible 4.9.0 Using pip3

Install the required Ansible version.

```bash
pip3 install ansible==4.9.0
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `pip3 install` | Installs Python packages |
| `ansible==4.9.0` | Installs the exact Ansible version required |

This installs:

```text
Ansible 4.9.0
```

along with the compatible Ansible Core package.

---

### Verify Installation Path

Check where the Ansible executable has been installed.

```bash
which ansible
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `which ansible` | Displays the path of the Ansible binary |

Example Output:

```text
/usr/local/bin/ansible
```

---

### Configure Global Access

Ensure the Ansible binary directory is included in the global PATH.

Open the global profile configuration file:

```bash
vi /etc/profile
```

Add the following line if `/usr/local/bin` is not already present:

```bash
export PATH=$PATH:/usr/local/bin
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `vi /etc/profile` | Opens the global environment configuration file |
| `export PATH=$PATH:/usr/local/bin` | Adds Ansible binary directory to the system PATH |

This makes Ansible accessible to all users on the system.

---

### Reload Environment Variables

Apply the updated PATH configuration without rebooting.

```bash
source /etc/profile
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `source /etc/profile` | Reloads environment variables immediately |

---

## Verification

### Verify Installed Version

Confirm that Ansible has been installed successfully.

```bash
ansible --version
```

Expected Output:

```text
ansible [core 2.11.x]
  config file = None
  python version = 3.x.x
```

Installed package version:

```text
ansible 4.9.0
```

---

### Verify Global Accessibility

Switch to another user and verify that Ansible is accessible without using a full path.

```bash
su - thor
```

```bash
ansible --version
```

Expected Result:

```text
ansible [core 2.11.x]
```

If the command executes successfully, Ansible is globally accessible.

---

## Workflow

```text
┌─────────────────────┐
│     Jump Host       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Python3      │
│ and pip3            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Install Ansible     │
│ Version 4.9.0       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Locate Ansible      │
│ Binary Path         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Update Global PATH  │
│ /etc/profile        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Reload Environment  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Installation │
│ ansible --version   │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `python3 --version` | Verify Python installation |
| `pip3 --version` | Verify pip3 installation |
| `pip3 install ansible==4.9.0` | Install Ansible 4.9.0 |
| `which ansible` | Locate Ansible executable |
| `vi /etc/profile` | Edit global environment variables |
| `source /etc/profile` | Reload environment variables |
| `ansible --version` | Verify Ansible installation |

---

## Key Learnings

- Learned how to install Ansible using pip3.
- Understood the relationship between Ansible and Python.
- Configured global PATH variables for system-wide access.
- Verified Ansible installation and version information.
- Prepared a Linux server for automation and configuration management.

---

## Best Practices

- Install a specific Ansible version to ensure consistency across environments.
- Verify Python and pip dependencies before installation.
- Keep `/usr/local/bin` available in the system PATH.
- Validate installations immediately after deployment.
- Use version-controlled automation tools across infrastructure environments.

---

## Challenge Status

✅ Successfully Completed
