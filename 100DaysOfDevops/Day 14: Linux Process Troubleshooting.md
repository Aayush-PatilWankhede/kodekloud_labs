# 📅 Day 14 - Apache Service Unavailability & Port Conflict Resolution

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)
![Service](https://img.shields.io/badge/Service-Apache%20HTTPD-red)
![Challenge](https://img.shields.io/badge/100--Days--AWS--DevOps--Challenge-Day%2014-blue)

---

## 📖 Challenge Overview

A monitoring tool raised an alert that Apache was unavailable on one of the application servers in Stratos DC. This challenge involved auditing all three app servers, identifying which host had a faulty Apache configuration, diagnosing the root cause (port conflict with `sendmail`), and restoring Apache to a healthy running state on port `5001` across the entire fleet — without disrupting any existing web content.

---

## 🎯 Objective

- Audit all three app servers for Apache (`httpd`) service health.
- Identify the faulty host where Apache is stopped or misconfigured.
- Ensure Apache is running and correctly bound to port `5001` on all servers.
- Resolve any port conflicts preventing Apache from starting.
- Confirm the fix is persistent across reboots on every app host.

---

## 🛠️ Technologies / Services Used

| Category      | Technology                  | Purpose                                          |
|---------------|-----------------------------|--------------------------------------------------|
| OS            | Linux (RHEL/CentOS family)  | Host operating system                            |
| Web Server    | Apache (httpd)              | Application service that must run on port 5001   |
| Conflicting Service | sendmail              | Was occupying port 5001, blocking Apache startup |
| Tools         | SSH, ss, grep, curl, systemctl | Diagnosis, configuration, and verification    |

---

## 📋 Challenge Requirements

- Target hosts: `stapp01`, `stapp02`, `stapp03`.
- Apache (`httpd`) must be running on **all three** app servers.
- Apache must be listening on port `5001` on all hosts.
- Fixes must be boot-persistent (`systemctl enable`).
- Existing web content must not be modified.

---

## 🎯 Solution Approach

A systematic audit was performed on all three app servers one by one — checking service state and port binding on each host. The root cause on the faulty server was a `sendmail` service occupying port `5001`, which prevented Apache from binding to it at startup. The fix involved stopping and disabling `sendmail`, verifying the Apache `Listen` directive was set to `5001`, then starting and enabling Apache. The same health checks were then confirmed on all remaining servers.

---

## 🚀 Implementation Steps

### Step 1: Audit Apache on All App Servers

Connected to each app server from the jump host and checked the Apache service state and port binding.

**App Server 1:**

```bash
ssh stapp01
sudo su -
systemctl status httpd
ss -tlnp | grep httpd
exit
exit
```

**App Server 2:**

```bash
ssh stapp02
sudo su -
systemctl status httpd
ss -tlnp | grep httpd
exit
exit
```

**App Server 3:**

```bash
ssh stapp03
sudo su -
systemctl status httpd
ss -tlnp | grep httpd
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl status httpd` | Shows whether Apache is active, failed, or stopped, along with recent log output |
| `ss -tlnp` | Lists all listening TCP sockets with associated process names |
| `grep httpd` | Filters socket output to show only Apache-related entries |

**Finding:** One server showed Apache in a `failed` or `stopped` state. The port diagnostic revealed `sendmail` was occupying port `5001`, preventing Apache from binding.

---

### Step 2: Identify the Root Cause — Port Conflict

On the faulty server, confirmed which process was holding port `5001`.

```bash
ss -tlnp | grep 5001
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `ss` | Displays socket statistics |
| `-t` | Shows TCP sockets only |
| `-l` | Shows listening sockets only |
| `-n` | Displays port numbers numerically |
| `-p` | Shows the process name and PID using each socket |
| `grep 5001` | Filters to show only port 5001 activity |

**Output:**

```text
127.0.0.1:5001   users:(("sendmail"...))
```

`sendmail` was occupying port `5001`, making it unavailable for Apache to bind.

---

### Step 3: Verify Apache Port Configuration

Confirmed Apache was configured to listen on port `5001`.

```bash
grep -n "^Listen" /etc/httpd/conf/httpd.conf
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `grep` | Searches file content for matching patterns |
| `-n` | Displays the line number of each match |
| `"^Listen"` | Anchors the search to lines that start with `Listen` |
| `/etc/httpd/conf/httpd.conf` | Apache's primary configuration file |

**Expected Output:**

```text
Listen 5001
```

If the directive showed a different port, edit the configuration:

```bash
vi /etc/httpd/conf/httpd.conf
```

Set the correct directive:

```text
Listen 5001
```

Save and exit:

```text
ESC → :wq → Enter
```

---

### Step 4: Stop and Disable the Conflicting sendmail Service

Freed port `5001` by stopping `sendmail` and preventing it from restarting on boot.

```bash
systemctl stop sendmail
systemctl disable sendmail
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl stop sendmail` | Immediately stops the `sendmail` service, releasing port 5001 |
| `systemctl disable sendmail` | Removes `sendmail` from the boot sequence, preventing the conflict from recurring after a reboot |

---

### Step 5: Start and Enable Apache

```bash
systemctl start httpd
systemctl enable httpd
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl start httpd` | Starts the Apache service immediately now that port 5001 is free |
| `systemctl enable httpd` | Ensures Apache starts automatically on every future boot |

---

### Step 6: Verify Apache Is Running on Port 5001

```bash
ss -tlnp | grep 5001
```

**Expected Output:**

```text
LISTEN  0  511  *:5001  *:*  users:(("httpd"...))
```

> ⚠️ If output still shows `127.0.0.1:5001`, Apache is bound to localhost only. Ensure the `Listen` directive in `httpd.conf` reads `Listen 5001` (not `Listen 127.0.0.1:5001`) and restart Apache.

---

### Step 7: Confirm Apache Service Health

```bash
systemctl status httpd
```

**Expected Output:**

```text
Active: active (running)
```

---

### Step 8: Repeat Audit and Fixes on All Remaining Servers

Return to the jump host and verify the same healthy state on the other two app servers:

```bash
exit
```

Repeat Steps 1–7 for any remaining server that is not yet confirmed healthy. All three hosts must satisfy:

- Apache status: `active (running)`
- Port binding: `*:5001`

---

## ✅ Verification

### Local Verification on Each App Server

```bash
curl http://localhost:5001
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `curl` | Sends an HTTP GET request |
| `http://localhost:5001` | Tests Apache from within the server itself on the required port |

**Expected Output:** An HTML response (even a default Apache page) confirms the service is up and reachable.

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `systemctl status httpd` | Check Apache service state |
| `ss -tlnp \| grep 5001` | Identify which process is bound to port 5001 |
| `grep -n "^Listen" /etc/httpd/conf/httpd.conf` | Verify Apache's configured listening port |
| `systemctl stop sendmail` | Stop the conflicting sendmail service |
| `systemctl disable sendmail` | Prevent sendmail from starting on boot |
| `systemctl start httpd` | Start Apache |
| `systemctl enable httpd` | Enable Apache to start on boot |
| `curl http://localhost:5001` | Confirm Apache is serving on the correct port |

---

## 🏗️ Architecture / Workflow

```text
Jump Host
   │
   ├──▶ stapp01 ──▶ Check Apache status & port
   ├──▶ stapp02 ──▶ Check Apache status & port
   └──▶ stapp03 ──▶ Check Apache status & port (FAULTY)
                           │
                           ▼
                  sendmail occupying :5001
                           │
                    systemctl stop sendmail
                    systemctl disable sendmail
                           │
                           ▼
                  httpd.conf → Listen 5001 ✅
                           │
                    systemctl start httpd
                    systemctl enable httpd
                           │
                           ▼
              Apache active (running) on *:5001 ✅
```

---

## 💡 Key Learnings

- Always audit all hosts in a fleet before assuming a single point of failure — the monitoring alert may not reveal which specific host is affected.
- `ss -tlnp | grep <port>` is the fastest way to identify which process is occupying a port on a RHEL/CentOS system.
- A `sendmail` service binding to a non-standard port (5001) is unusual — this often results from a misconfigured or repurposed sendmail setup that wasn't cleaned up.
- Disabling a conflicting service (`systemctl disable`) is as important as stopping it — without disabling, the conflict will reappear on next reboot.
- `Listen 5001` in `httpd.conf` binds Apache to all interfaces; `Listen 127.0.0.1:5001` restricts it to localhost — always verify the binding scope.
- Verifying with `curl http://localhost:5001` on each host independently is more reliable than assuming a fleet-wide fix worked uniformly.

---

## 🔒 Best Practices

- Audit all hosts systematically rather than only targeting the first server that appears faulty.
- Always check what process is holding a port before stopping it — confirm the conflict before taking action.
- Stop **and** disable conflicting services to prevent recurrence after a reboot.
- Validate the Apache `Listen` directive explicitly before restarting, to avoid restarting into an already-known bad config.
- Enable Apache at boot (`systemctl enable`) on every host, not just the one that was actively broken.
- Confirm the fix on each host independently using a local `curl` test rather than assuming a uniform outcome.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
