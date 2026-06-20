# 📅 Day 13 - Restricting Apache Port Access Using iptables (Load Balancer Whitelisting)

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)
![Security](https://img.shields.io/badge/Security-iptables-red)
![Challenge](https://img.shields.io/badge/100--Days--AWS--DevOps--Challenge-Day%2013-blue)

---

## 📖 Challenge Overview

The Nautilus security team identified that Apache's service port (`8083`) was openly accessible to anyone on the network across all app hosts in Stratos DC, since no firewall was installed. This challenge addresses that exposure by installing `iptables`, restricting inbound access on port `8083` to only the load balancer host, and ensuring the rules persist across reboots — reducing the attack surface while preserving legitimate traffic flow through the load balancer.

---

## 🎯 Objective

- Install `iptables` and its required dependencies on every app host.
- Block inbound access to port `8083` for all sources **except** the load balancer (LB) host.
- Ensure firewall rules survive a system reboot.
- Apply the same configuration consistently across all three app servers.

---

## 🛠️ Technologies / Services Used

| Category     | Technology | Purpose |
|---------------|------------|---------|
| OS            | Linux (RHEL/CentOS family) | Host operating system |
| Web Server    | Apache (httpd) | Application service running on port 8083 |
| Security      | iptables, iptables-services | Firewall enforcement and rule persistence |
| Infrastructure| Load Balancer (`stlb01`) | Sole permitted source for port 8083 traffic |
| Tools         | SSH, getent, ss | Connectivity, DNS resolution, and port verification |

---

## 📋 Challenge Requirements

- Target hosts: `stapp01`, `stapp02`, `stapp03`.
- Install `iptables` and `iptables-services` on each host.
- Allow inbound TCP traffic on port `8083` only from the load balancer's IP.
- Drop inbound TCP traffic on port `8083` from all other sources.
- Persist firewall rules so they remain active after a reboot.

---

## 🎯 Solution Approach

The same hardening procedure was applied identically across all three app servers: install the firewall packages, resolve the load balancer's IP dynamically via DNS, confirm Apache was correctly listening on port 8083, then insert two ordered `iptables` rules — an `ACCEPT` rule scoped to the load balancer's IP followed by a `DROP` rule for all other sources on that port. Rules were then persisted to disk so they survive a reboot, and verified both locally and from the load balancer/jump host perspective.

---

## 🚀 Implementation Steps

> The steps below were executed identically on `stapp01`, `stapp02`, and `stapp03`.

### Step 1: Connect to the App Server and Elevate Privileges

```bash
ssh stapp01
sudo su -
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `ssh stapp01` | Connects to the target application server |
| `sudo su -` | Elevates to root with a full login environment, required for firewall and package changes |

---

### Step 2: Install iptables and Dependencies

```bash
yum install -y iptables iptables-services
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `yum install` | Installs the specified packages |
| `-y` | Auto-confirms the installation prompt |
| `iptables` | Core firewall utility for managing kernel netfilter rules |
| `iptables-services` | Provides the systemd unit and `service iptables save` persistence mechanism |

---

### Step 3: Enable and Start the iptables Service

```bash
systemctl enable iptables
systemctl start iptables
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl enable iptables` | Ensures the firewall starts automatically on every boot |
| `systemctl start iptables` | Starts the firewall service immediately |

---

### Step 4: Resolve the Load Balancer's IP Address

```bash
getent hosts stlb01
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `getent` | Queries system databases, including DNS/hosts resolution |
| `hosts stlb01` | Resolves the IP address for the `stlb01` load balancer hostname |

**Example Output:**

```text
10.244.30.27   stlb01
```

This IP is used as the sole permitted source for port 8083 in the next steps.

---

### Step 5: Confirm Apache Is Listening on Port 8083

```bash
systemctl status httpd
ss -tlnp | grep 8083
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `systemctl status httpd` | Confirms Apache is running |
| `ss -tlnp` | Lists listening TCP sockets with the owning process |
| `grep 8083` | Filters output to the relevant port |

**Expected Output:**

```text
LISTEN  0  511  *:8083  *:*  users:(("httpd"...))
```

If Apache isn't running or bound correctly:

```bash
systemctl start httpd
systemctl enable httpd
grep -n "^Listen" /etc/httpd/conf/httpd.conf
```

Set the directive to `Listen 8083` if missing, then restart:

```bash
systemctl restart httpd
```

---

### Step 6: Insert the Firewall Rules

> ⚠️ Before inserting rules by position number, check the existing chain first — rule numbering depends on what's already present on the host:
> ```bash
> iptables -L INPUT --line-numbers
> ```
> Insert the `ACCEPT` rule immediately before any existing catch-all `DROP`/`REJECT` rule, and the new `DROP` rule directly after it, so the LB-specific `ACCEPT` is always evaluated first.

Allow the load balancer's IP on port 8083:

```bash
iptables -I INPUT 5 -p tcp -s 10.244.30.27 --dport 8083 -j ACCEPT
```

Block everyone else on port 8083:

```bash
iptables -I INPUT 6 -p tcp --dport 8083 -j DROP
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `-I INPUT 5` / `-I INPUT 6` | Inserts the rule at a specific position in the INPUT chain, ensuring correct evaluation order |
| `-p tcp` | Matches TCP traffic only |
| `-s 10.244.30.27` | Restricts the ACCEPT rule to traffic sourced from the load balancer's IP |
| `--dport 8083` | Matches traffic destined for Apache's port |
| `-j ACCEPT` / `-j DROP` | Action taken on a match — allow or silently discard the traffic |

**Result:**
- Traffic from the load balancer on port 8083 → **allowed**.
- Traffic from any other source on port 8083 → **dropped**.

---

### Step 7: Persist the Rules

```bash
iptables-save > /etc/sysconfig/iptables
systemctl restart iptables
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `iptables-save > /etc/sysconfig/iptables` | Writes the current rule set to disk so it survives a reboot |
| `systemctl restart iptables` | Reloads the firewall from the saved file, doubling as a persistence sanity check |

---

### Step 8: Verify Rule Order

```bash
iptables -L -n
```

**Expected Order (relevant excerpt):**

| Target | Protocol | Source | Destination Port |
|--------|----------|--------|-------------------|
| ACCEPT | tcp | `10.244.30.27` (LB) | `8083` |
| DROP   | tcp | `0.0.0.0/0` (everyone) | `8083` |

The `ACCEPT` rule for the LB must appear **before** the `DROP` rule — `iptables` evaluates rules top-down and stops at the first match.

---

### Step 9: Repeat on the Remaining App Servers

Apply Steps 1–8 identically on:

```bash
ssh stapp02
```

```bash
ssh stapp03
```

---

## ✅ Verification

### From the Load Balancer Host (Should Succeed)

```bash
curl http://stapp01:8083
curl http://stapp02:8083
curl http://stapp03:8083
```

**Expected Output:** Apache's HTML page is returned from all three hosts.

### From the Jump Host (Should Be Blocked)

```bash
curl http://stapp01:8083
```

**Expected Output:** The request times out or is refused, confirming the firewall is correctly restricting access to only the load balancer's IP.

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `yum install -y iptables iptables-services` | Install firewall and persistence support |
| `systemctl enable iptables && systemctl start iptables` | Enable and start the firewall |
| `getent hosts <hostname>` | Resolve a hostname to its IP address |
| `ss -tlnp \| grep <port>` | Verify a service is listening on a given port |
| `iptables -L INPUT --line-numbers` | View rule positions before inserting new ones |
| `iptables -I INPUT <pos> -p tcp -s <ip> --dport <port> -j ACCEPT` | Allow traffic from a specific source |
| `iptables -I INPUT <pos> -p tcp --dport <port> -j DROP` | Block traffic from all other sources |
| `iptables-save > /etc/sysconfig/iptables` | Persist rules across reboots |
| `iptables -L -n` | List active rules numerically |

---

## 🏗️ Architecture / Workflow

```text
                 ┌────────────────────┐
                 │   Load Balancer    │
                 │     (stlb01)       │
                 └─────────┬──────────┘
                           │ ACCEPT :8083
                           ▼
   ┌───────────────────────────────────────────┐
   │            iptables Firewall                │
   │  ACCEPT  10.244.30.27 → :8083               │
   │  DROP    0.0.0.0/0     → :8083               │
   └───────────────────────────────────────────┘
                           │
                           ▼
        ┌─────────┐   ┌─────────┐   ┌─────────┐
        │ stapp01 │   │ stapp02 │   │ stapp03 │
        │ :8083   │   │ :8083   │   │ :8083   │
        └─────────┘   └─────────┘   └─────────┘

   Jump Host / Any Other Source ──── ✕ DROPPED ──── :8083
```

---

## 💡 Key Learnings

- `iptables` evaluates rules sequentially top-down — the first matching rule wins, so an `ACCEPT` rule must always precede a broader `DROP`/`REJECT` rule covering the same traffic.
- Rule insertion position (`-I INPUT <n>`) is relative to the host's *existing* rule set — always check with `--line-numbers` rather than assuming a fixed position across different hosts.
- `iptables-services` is required for the `service iptables save` / boot-time restore workflow on RHEL-family systems; raw `iptables` alone does not persist rules.
- `getent hosts` is a clean, script-friendly way to resolve a hostname's IP without parsing `ping` or `nslookup` output.
- Verifying from multiple vantage points (LB vs. jump host) is the only reliable way to confirm a source-restricted firewall rule is actually working as intended.

---

## 🔒 Best Practices

- Always verify existing firewall rules (`--line-numbers`) before inserting new ones by position, to avoid misordering or accidental lockout.
- Whitelist by specific source IP (`-s <ip>`) rather than opening a port broadly, following the principle of least privilege.
- Persist firewall changes immediately after applying them — an unsaved rule is lost on the next reboot.
- Apply identical, repeatable procedures across all hosts in a fleet to avoid configuration drift.
- Always test from both an allowed and a disallowed source to confirm a security rule behaves as expected in both directions.
- Take care not to inadvertently block SSH (port 22) or other essential management ports while inserting firewall rules.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
