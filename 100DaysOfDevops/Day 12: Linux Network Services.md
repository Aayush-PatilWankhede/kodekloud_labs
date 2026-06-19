# 📅 Day 12 - Apache Service Unreachable on Custom Port (Troubleshooting)

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)
![Service](https://img.shields.io/badge/Service-Apache%20HTTPD-red)
![Challenge](https://img.shields.io/badge/100--Days--DevOps--Challenge-Day%2012-blue)

---

## 📖 Challenge Overview

A monitoring tool flagged an issue in the Stratos Datacenter: an application server's Apache service was unreachable on its configured port (3004). This challenge involved performing structured network and service-level troubleshooting to diagnose whether the root cause was the Apache service itself, a port conflict, or a firewall restriction — and resolving it without modifying the existing web application code.

---

## 🛠️ Technologies Used

| Category   | Technology                          |
|------------|--------------------------------------|
| OS         | Linux (RHEL/CentOS family)           |
| Web Server | Apache (httpd)                       |
| Networking | TCP/IP, iptables                     |
| Tools      | SSH, curl, telnet, ss, systemctl     |

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

**Finding:** Apache failed to start with the error `(98) Address already in use`, indicating a port conflict rather than a configuration failure.

---

### Step 3: Verify Apache's Configured Listening Port

Confirmed Apache was correctly configured to use port 3004.

```bash
grep -n "Listen" /etc/httpd/conf/httpd.conf
```

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

---

### Step 6: Validate Configuration and Start Apache

Verified the Apache configuration syntax before restarting the service.

```bash
httpd -t
systemctl restart httpd
systemctl status httpd
```

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

This returned the HTML page successfully, confirming Apache was healthy internally.

---

### Step 8: Diagnose and Resolve the Remaining Firewall Block

Despite Apache running correctly, the jump host still received `No route to host` — pointing to a firewall-level block. Since `firewalld` was not installed on this host, `iptables` was used directly.

```bash
iptables -L -n
```

Added a rule to permit inbound TCP traffic on port 3004:

```bash
iptables -I INPUT -p tcp --dport 3004 -j ACCEPT
```

Persisted the rule:

```bash
service iptables save
```

---

### Step 9: Final Validation from Jump Host

```bash
curl http://stapp01:3004
```

Apache's HTML page was returned successfully, confirming the issue was fully resolved.

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `systemctl status httpd` | Check Apache service status |
| `ss -tulpn \| grep 3004` | Identify which process is using a port |
| `httpd -t` | Validate Apache configuration syntax |
| `systemctl restart httpd` | Restart the Apache service |
| `iptables -L -n` | List current firewall rules |
| `iptables -I INPUT -p tcp --dport 3004 -j ACCEPT` | Allow inbound traffic on a specific port |
| `curl http://stapp01:3004` | Test external reachability of the Apache service |

---

## ✅ Verification

### Verify Apache Is Reachable from the Jump Host

```bash
curl http://stapp01:3004
```

**Expected Output:**

```text
<HTML content of the existing index.html page>
```

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

---

## 🔒 Best Practices

- Diagnose layer by layer: service → port binding → firewall, rather than guessing at the root cause.
- Avoid killing unknown processes blindly — identify and disable the correct service (`sendmail`) rather than force-killing the port.
- Only open the specific port required (`3004`) in the firewall rather than broadly disabling firewall protections.
- Persist firewall rule changes so fixes survive a reboot.
- Always verify a fix from the perspective of the original failure point (the jump host), not just locally on the server.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
