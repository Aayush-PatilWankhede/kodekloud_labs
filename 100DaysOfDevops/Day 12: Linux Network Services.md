# 📅 Day 12 - Apache Service Unreachable on Custom Port (Troubleshooting)

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)
![Service](https://img.shields.io/badge/Service-Apache%20HTTPD-red)
![Challenge](https://img.shields.io/badge/100--Days--AWS--DevOps--Challenge-Day%2012-blue)

---

## 📖 Challenge Overview

A monitoring tool flagged an issue in the Stratos Datacenter: an application server's Apache service was unreachable on its configured port (3004). This challenge involved performing structured network and service-level troubleshooting to diagnose whether the root cause was the Apache service itself, a port conflict, or a firewall restriction — and resolving it without modifying the existing web application code.

---

## 🎯 Objective

- Diagnose why Apache on `stapp01` was unreachable on port `3004`.
- Use diagnostic tools (`telnet`, `ss`, etc.) to isolate the root cause.
- Fix the issue without altering the existing `index.html` content.
- Ensure Apache is reachable from the jump host without weakening security posture.
- Validate the fix using `curl http://stapp01:3004` from the jump host.

---

## 🛠️ Technologies / Services Used

| Category   | Technology | Purpose |
|------------|------------|---------|
| OS         | Linux (RHEL/CentOS family) | Host operating system |
| Web Server | Apache (httpd) | Serves the application on port 3004 |
| Networking | TCP/IP, iptables | Connectivity and traffic filtering |
| Tools      | SSH, curl, telnet, ss, systemctl, journalctl | Diagnosis and remediation |

---

## 📋 Challenge Requirements

- Diagnose why Apache on `stapp01` was unreachable on port `3004`.
- Use diagnostic tools (`telnet`, `netstat`/`ss`, etc.) to isolate the root cause.
- Fix the issue without altering the existing `index.html` content.
- Ensure Apache is reachable from the jump host without weakening security posture.
- Validate the fix using `curl http://stapp01:3004` from the jump host.

---

## 🎯 Solution Approach

The troubleshooting followed a layered diagnostic methodology — starting from the network layer (jump host connectivity), moving to the service layer (Apache process and port binding), and finally the security layer (firewall rules). This systematic approach isolated three distinct issues: a port conflict with `sendmail`, an Apache binding configuration, and a firewall rule blocking external access — each resolved independently and verified before moving to the next.

---

## 🚀 Implementation Steps

### Step 1: Confirm the Connectivity Issue from the Jump Host

Reproduced the reported issue to confirm the failure before making any changes.

```bash
curl http://stapp01:3004
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `curl` | Sends an HTTP request to test service reachability |
| `http://stapp01:3004` | Target host and port being tested |

**Result:**

```text
curl: (7) Failed to connect to stapp01 port 3004: No route to host
```

This narrowed the cause to either the Apache service, port misconfiguration, or a firewall block.

---

### Step 2: Check Apache Service Status on the App Server

Logged into the affected server and inspected the Apache (`httpd`) service state.

```bash
ssh stapp01
sudo su -
systemctl status httpd
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `ssh stapp01` | Connects to the affected application server |
| `sudo su -` | Elevates to root with a full login environment |
| `systemctl status httpd` | Displays current state and recent log lines for the Apache service |

**Finding:** Apache failed to start with the error `(98) Address already in use`, indicating a port conflict rather than a configuration failure.

> 💡 If the bind error doesn't appear in `systemctl status`, check the full service journal:
> ```bash
> journalctl -xeu httpd
> ```
> | Option | Description |
> |--------|-------------|
> | `-x` | Adds explanatory help text for log entries where available |
> | `-e` | Jumps to the end (most recent) entries |
> | `-u httpd` | Filters logs to only the `httpd` unit |

---

### Step 3: Verify Apache's Configured Listening Port

Confirmed Apache was correctly configured to use port 3004.

```bash
grep -n "Listen" /etc/httpd/conf/httpd.conf
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `grep` | Searches file content for matching text |
| `-n` | Prefixes matches with their line number |
| `"Listen"` | Search term — the directive controlling Apache's bind port |
| `/etc/httpd/conf/httpd.conf` | Apache's main configuration file |

**Output:**

```text
Listen 3004
```

---

### Step 4: Identify the Process Occupying Port 3004

Used `ss` to find which process was holding the port before Apache could bind to it.

```bash
ss -tulpn | grep 3004
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `ss` | Displays active socket connections |
| `-t` | Shows TCP sockets |
| `-u` | Shows UDP sockets |
| `-l` | Shows only listening sockets |
| `-p` | Shows the process using each socket |
| `-n` | Displays numeric ports instead of resolving service names |
| `grep 3004` | Filters output to only port 3004 |

**Output:**

```text
127.0.0.1:3004   users:(("sendmail"...))
```

**Root Cause #1:** The `sendmail` service was occupying port 3004, preventing Apache from starting.

---

### Step 5: Stop and Disable the Conflicting Service

Freed the port by stopping `sendmail` and disabling it from future startups.

```bash
systemctl stop sendmail
systemctl disable sendmail
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl stop sendmail` | Immediately stops the running `sendmail` service, releasing port 3004 |
| `systemctl disable sendmail` | Prevents `sendmail` from auto-starting on future boots, avoiding the conflict from recurring |

---

### Step 6: Validate Configuration and Start Apache

Verified the Apache configuration syntax before restarting the service.

```bash
httpd -t
systemctl restart httpd
systemctl status httpd
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `httpd -t` | Tests the Apache configuration for syntax errors without restarting the service |
| `systemctl restart httpd` | Stops and starts Apache to apply the now-available port binding |
| `systemctl status httpd` | Confirms the service is active and running |

**Expected Output:**

```text
Syntax OK
active (running)
```

---

### Step 7: Verify Apache Is Listening on All Interfaces

Confirmed Apache was bound externally and not restricted to localhost.

```bash
ss -tulpn | grep 3004
```

**Correct Output:**

```text
*:3004
```

> ⚠️ If the output showed `127.0.0.1:3004` instead, Apache would only be reachable locally. The `Listen 3004` directive (not `Listen 127.0.0.1:3004`) in `httpd.conf` ensures external accessibility.

Local validation:

```bash
curl http://localhost:3004
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `curl` | Sends an HTTP request |
| `http://localhost:3004` | Tests Apache from within the server itself, bypassing external network/firewall paths |

This returned the HTML page successfully, confirming Apache was healthy internally.

---

### Step 8: Diagnose and Resolve the Remaining Firewall Block

Despite Apache running correctly, the jump host still received `No route to host` — pointing to a firewall-level block. Since `firewalld` was not installed on this host, `iptables` was used directly.

```bash
iptables -L -n
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `iptables` | Manages Linux kernel firewall rules |
| `-L` | Lists all current rules |
| `-n` | Displays IP addresses and ports numerically (skips DNS/service-name resolution for faster, clearer output) |

Added a rule to permit inbound TCP traffic on port 3004:

```bash
iptables -I INPUT -p tcp --dport 3004 -j ACCEPT
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `-I INPUT` | Inserts the rule at the top of the INPUT chain, so it's evaluated before any later DROP/REJECT rule |
| `-p tcp` | Matches TCP traffic only |
| `--dport 3004` | Matches traffic destined for port 3004 |
| `-j ACCEPT` | Action to take on a match — allow the traffic |

Persisted the rule so it survives a service/reboot:

```bash
service iptables save
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `service iptables save` | Writes the active rule set to disk (requires the `iptables-services` package) |

> 💡 If `iptables-services` isn't installed, save manually instead:
> ```bash
> iptables-save > /etc/sysconfig/iptables
> ```
> Also ensure rules are restored on boot:
> ```bash
> systemctl enable iptables
> ```

---

### Step 9: Final Validation from Jump Host

```bash
curl http://stapp01:3004
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `curl` | Re-runs the original reachability test |
| `http://stapp01:3004` | Confirms the service is now externally accessible end-to-end |

Apache's HTML page was returned successfully, confirming the issue was fully resolved.

---

## ✅ Verification

### Verify Apache Is Reachable from the Jump Host

```bash
curl http://stapp01:3004
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `curl` | Sends an HTTP GET request |
| `http://stapp01:3004` | Confirms reachability over the network on the resolved port |

**Expected Output:**

```text
<HTML content of the existing index.html page>
```

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `systemctl status httpd` | Check Apache service status |
| `journalctl -xeu httpd` | View detailed service logs/errors |
| `ss -tulpn \| grep 3004` | Identify which process is using a port |
| `httpd -t` | Validate Apache configuration syntax |
| `systemctl restart httpd` | Restart the Apache service |
| `iptables -L -n` | List current firewall rules |
| `iptables -I INPUT -p tcp --dport 3004 -j ACCEPT` | Allow inbound traffic on a specific port |
| `iptables-save > /etc/sysconfig/iptables` | Persist firewall rules without `iptables-services` |
| `curl http://stapp01:3004` | Test external reachability of the Apache service |

---

## 🏗️ Architecture / Workflow

```text
Jump Host
   │  curl http://stapp01:3004
   ▼
Firewall (iptables)
   │  Port 3004 ACCEPT rule
   ▼
App Server (stapp01)
   │  Port freed from sendmail conflict
   ▼
Apache (httpd) Service
   │  Listening on *:3004
   ▼
index.html Served Successfully
```

---

## 💡 Key Learnings

- An `(98) Address already in use` error means another process is already bound to the target port — check before assuming a configuration fault.
- `ss -tulpn` is one of the most reliable tools for diagnosing port-binding conflicts.
- A listening address of `*:3004` or `0.0.0.0:3004` means the service accepts external connections; `127.0.0.1:3004` restricts it to localhost only.
- `No route to host` errors, even when a service is healthy, frequently point to a firewall rule rather than the application itself.
- Always validate configuration syntax (`httpd -t`) before restarting a service in a production-like environment.
- Not all systems use `firewalld` — some rely on raw `iptables`, and troubleshooting should adapt accordingly.
- Saved firewall rules only survive a reboot if the `iptables` service is also enabled at boot.

---

## 🔒 Best Practices

- Diagnose layer by layer: service → port binding → firewall, rather than guessing at the root cause.
- Avoid killing unknown processes blindly — identify and disable the correct service (`sendmail`) rather than force-killing the port.
- Only open the specific port required (`3004`) in the firewall rather than broadly disabling firewall protections.
- Persist firewall rule changes **and** enable the firewall service so fixes survive a reboot.
- Always verify a fix from the perspective of the original failure point (the jump host), not just locally on the server.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
