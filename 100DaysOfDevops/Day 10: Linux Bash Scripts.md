# Day 10 - Create a Bash Script to Archive Website Content

## Overview

This challenge focuses on automating website content backups using a Bash script. The objective is to create a script that archives the website content hosted on App Server 2, stores the archive locally, and securely copies it to the Nautilus Storage Server for long-term retention.

The solution uses the `zip` utility for archive creation and `scp` with SSH key-based authentication for password-less file transfer.

---

## Objective

- Install the required `zip` package on App Server 2
- Create a Bash script named `blog_archive.sh`
- Archive the `/var/www/html/blog` directory
- Store the archive locally under `/archives`
- Copy the archive to the Nautilus Storage Server
- Configure password-less SSH authentication
- Ensure the script can be executed by the respective server user
- Avoid using `sudo` inside the script

---

## Services & Tools Used

| Service / Tool | Purpose |
|---------------|---------|
| Linux | Operating system |
| Bash | Automation scripting |
| zip | Create compressed archives |
| SCP | Secure file transfer |
| SSH | Password-less authentication |
| Storage Server | Long-term archive storage |

---

## Implementation

### Install Required Package

The task requires the `zip` package to be installed manually before executing the script.

```bash
yum install -y zip
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `yum install -y zip` | Installs the zip utility required for archive creation |

---

### Create Required Directories

Create the directories required for storing the script and archive files.

```bash
mkdir -p /scripts
mkdir -p /archives
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `mkdir -p /scripts` | Creates the script storage directory |
| `mkdir -p /archives` | Creates the local archive storage directory |
| `-p` | Creates parent directories if they do not exist |

---

### Configure Password-less SSH Authentication

Generate an SSH key pair for the application server user.

```bash
ssh-keygen -t rsa
```

Press **Enter** for all prompts.

#### Command Explanation

| Command | Description |
|----------|-------------|
| `ssh-keygen` | Generates SSH public/private key pair |
| `-t rsa` | Uses RSA encryption algorithm |

---

### Copy Public Key to Storage Server

Configure password-less authentication to the Storage Server.

```bash
ssh-copy-id natasha@ststor01
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `ssh-copy-id` | Copies the public SSH key to the remote server |
| `natasha@ststor01` | Storage server user and hostname |

---

### Verify Password-less Access

Verify that SSH login works without a password.

```bash
ssh natasha@ststor01
```

Exit the server after verification:

```bash
exit
```

---

### Create the Backup Script

Create the script in the required location.

```bash
vi /scripts/blog_archive.sh
```

Add the following content:

```bash
#!/bin/bash

zip -r /archives/xfusioncorp_blog.zip /var/www/html/blog

scp /archives/xfusioncorp_blog.zip natasha@ststor01:/archives/
```

Save and exit the editor.

#### Script Explanation

| Command | Purpose |
|----------|----------|
| `zip -r` | Creates a recursive ZIP archive |
| `/archives/xfusioncorp_blog.zip` | Archive destination |
| `/var/www/html/blog` | Source website directory |
| `scp` | Securely copies files to another server |
| `natasha@ststor01:/archives/` | Storage server destination |

---

### Make the Script Executable

Grant execute permission to the script.

```bash
chmod +x /scripts/blog_archive.sh
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `chmod +x` | Adds execute permission to the script |

---

### Execute the Script

Run the script to create and transfer the archive.

```bash
/scripts/blog_archive.sh
```

Example Output:

```text
adding: var/www/html/blog/
adding: var/www/html/blog/index.html

xfusioncorp_blog.zip 100%
```

---

## Verification

### Verify Archive on App Server 2

```bash
ls -l /archives
```

Expected Output:

```text
xfusioncorp_blog.zip
```

---

### Verify Archive on Storage Server

```bash
ssh natasha@ststor01
```

```bash
ls -l /archives
```

Expected Output:

```text
xfusioncorp_blog.zip
```

This confirms that the archive has been successfully copied to the Storage Server.

---

## Script Content

```bash
#!/bin/bash

zip -r /archives/xfusioncorp_blog.zip /var/www/html/blog

scp /archives/xfusioncorp_blog.zip natasha@ststor01:/archives/
```

---

## Workflow

```text
┌────────────────────────┐
│ Website Content        │
│ /var/www/html/blog     │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ blog_archive.sh        │
│ Bash Script            │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ ZIP Archive Creation   │
│ xfusioncorp_blog.zip   │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ Local Storage          │
│ /archives              │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ SCP Transfer           │
│ Password-less SSH      │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ Nautilus Storage       │
│ ststor01:/archives     │
└────────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `yum install -y zip` | Install ZIP utility |
| `mkdir -p /scripts` | Create script directory |
| `mkdir -p /archives` | Create archive directory |
| `ssh-keygen -t rsa` | Generate SSH key pair |
| `ssh-copy-id` | Configure password-less SSH |
| `zip -r` | Create ZIP archive |
| `scp` | Securely copy files |
| `chmod +x` | Make script executable |
| `ls -l` | Verify file existence |

---

## Key Learnings

- Learned how to automate backup operations using Bash scripting.
- Practiced creating compressed archives with the `zip` utility.
- Configured password-less SSH authentication using RSA keys.
- Used SCP for secure file transfers between Linux servers.
- Implemented an automated archive workflow for website content.

---

## Best Practices

- Store scripts in dedicated directories such as `/scripts`.
- Use SSH key-based authentication instead of passwords.
- Keep archive copies in multiple locations for redundancy.
- Test backup scripts manually before scheduling them.
- Avoid using `sudo` inside automation scripts unless required.
- Regularly verify backup integrity and file transfers.

---

## Challenge Status

✅ Successfully Completed
