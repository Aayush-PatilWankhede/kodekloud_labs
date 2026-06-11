# Day 06 - Install Cronie and Configure Scheduled Cron Jobs

## Overview

This challenge focuses on automating routine administrative tasks using Cron. Cron is a time-based job scheduler in Linux that allows commands and scripts to run automatically at predefined intervals.

The objective is to install the `cronie` package on all application servers, ensure the `crond` service is running, and configure a cron job for the root user that executes every 5 minutes.

---

## Objective

- Install the `cronie` package on all application servers
- Enable and start the `crond` service
- Configure a cron job for the root user
- Schedule the job to run every 5 minutes
- Verify the cron configuration and execution

---

## Services Used

| Service | Purpose |
|----------|----------|
| Cronie | Linux cron scheduling package |
| crond | Cron daemon service |
| systemd | Service management framework |
| SSH | Remote server administration |
| Crontab | Manage scheduled tasks |

---

## Implementation

### Connect to the Target Server

Log in to the application server.

```bash
ssh tony@stapp01
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `ssh` | Secure Shell utility used to connect to remote systems |
| `tony` | User account used for login |
| `stapp01` | Target application server |

> Repeat the same procedure for `stapp02` and `stapp03`.

---

### Switch to the Root User

Obtain administrative privileges required for package installation and cron configuration.

```bash
sudo su -
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `sudo` | Execute commands with elevated privileges |
| `su` | Switch user account |
| `-` | Load the target user's environment |

---

### Install the Cronie Package

Install the cron scheduling package.

```bash
yum install -y cronie
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `yum install` | Installs packages from repositories |
| `-y` | Automatically confirms installation prompts |
| `cronie` | Cron daemon package used for job scheduling |

---

### Enable and Start the Cron Service

Configure the cron daemon to start automatically on boot and start it immediately.

```bash
systemctl enable crond
```

```bash
systemctl start crond
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `systemctl enable crond` | Enables the cron service at system boot |
| `systemctl start crond` | Starts the cron service immediately |

---

### Verify the Cron Service Status

Confirm that the cron daemon is running successfully.

```bash
systemctl status crond
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `systemctl status` | Displays the current service status |
| `crond` | Cron daemon service |

Expected Output:

```text
Active: active (running)
```

The important part is:

```text
active (running)
```

which confirms that the cron service is operational.

---

### Configure the Cron Job

Open the root user's crontab configuration.

```bash
crontab -e
```

Add the following entry:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

#### Command Explanation

| Component | Description |
|------------|-------------|
| `*/5` | Execute every 5 minutes |
| `*` | Every hour |
| `*` | Every day of the month |
| `*` | Every month |
| `*` | Every day of the week |
| `echo hello > /tmp/cron_text` | Writes the text "hello" to the file |

Save and exit the editor.

```text
:wq
```

---

## Verification

### Verify the Configured Cron Job

Display all configured cron jobs for the current user.

```bash
crontab -l
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `crontab` | Manage scheduled cron jobs |
| `-l` | List configured cron entries |

Expected Output:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

---

### Verify Cron Job Execution

After waiting approximately 5 minutes, verify that the scheduled task has executed successfully.

```bash
cat /tmp/cron_text
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `cat` | Display file contents |
| `/tmp/cron_text` | File created by the cron job |

Expected Output:

```text
hello
```

The presence of this output confirms that the cron job is executing successfully.

---

## Cron Schedule Breakdown

```text
┌──────── Minute (0 - 59)
│ ┌────── Hour (0 - 23)
│ │ ┌──── Day of Month (1 - 31)
│ │ │ ┌── Month (1 - 12)
│ │ │ │ ┌ Day of Week (0 - 7)
│ │ │ │ │
│ │ │ │ │
* * * * *  command-to-execute
```

Cron Expression Used:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

Meaning:

```text
Run every 5 minutes
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
│ Connect to App      │
│ Server              │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Install Cronie      │
│ Package             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Enable & Start      │
│ crond Service       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Configure Root      │
│ Cron Job            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Execute Every       │
│ 5 Minutes           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create File         │
│ /tmp/cron_text      │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `yum install -y cronie` | Install Cronie package |
| `systemctl enable crond` | Enable cron service at boot |
| `systemctl start crond` | Start cron service |
| `systemctl status crond` | Verify cron service status |
| `crontab -e` | Edit cron jobs |
| `crontab -l` | List configured cron jobs |
| `cat /tmp/cron_text` | Verify cron job execution |

---

## Key Learnings

- Learned how to install and configure the Cronie package.
- Understood how the `crond` daemon manages scheduled tasks.
- Configured recurring jobs using crontab.
- Verified scheduled task execution through generated output files.
- Learned how cron expressions define execution schedules.

---

## Best Practices

- Use cron jobs for repetitive administrative tasks.
- Keep cron commands simple and well-documented.
- Verify cron execution through logs or output files.
- Run scheduled tasks with the minimum required privileges.
- Regularly review and clean up obsolete cron entries.

---

## Challenge Status

✅ Successfully Completed
