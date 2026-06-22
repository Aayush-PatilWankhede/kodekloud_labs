# 📅 Day 14 - Terminating an Obsolete EC2 Instance

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Cloud](https://img.shields.io/badge/Cloud-AWS-orange)
![Service](https://img.shields.io/badge/Service-EC2-yellow)
![Challenge](https://img.shields.io/badge/100--Days--AWS--DevOps--Challenge-Day%2014-blue)

---

## 📖 Challenge Overview

As part of an ongoing cloud migration, several AWS resources became obsolete after alternative solutions were implemented. Retaining unused EC2 instances increases cost and widens the attack surface unnecessarily. This challenge involves safely identifying and terminating a specific EC2 instance that is no longer in use, and confirming it reaches a fully `terminated` state before the task is considered complete.

---

## 🎯 Objective

- Locate the EC2 instance named `datacenter-ec2` in the `us-east-1` region.
- Terminate the instance cleanly using the AWS CLI.
- Confirm the instance reaches the `terminated` state before submitting.

---

## 🛠️ Technologies / Services Used

| Category | Technology | Purpose |
|----------|------------|---------|
| Cloud    | AWS        | Cloud infrastructure provider |
| Service  | EC2        | Compute instance being decommissioned |
| Tool     | AWS CLI    | Resource lookup and termination |

---

## 📋 Challenge Requirements

- Target instance: `datacenter-ec2` (existing EC2 instance in `us-east-1`).
- The instance must be in the `terminated` state before task completion.
- All actions must be scoped to the `us-east-1` region.
- Credentials retrieved securely via the `showcreds` utility on the AWS client host.

---

## 🎯 Solution Approach

The decommission followed a safe locate → terminate → verify workflow: first confirming the active CLI identity, then resolving the instance ID dynamically from its `Name` tag rather than hardcoding it, issuing the `terminate-instances` API call, and finally confirming the lifecycle state reached `terminated` — both via a blocking `wait` command and a final `describe-instances` check.

---

## 🚀 Implementation Steps

### Step 1: Verify AWS CLI Authentication

Confirmed the CLI session was authenticated against the correct AWS account before making any destructive changes.

```bash
aws sts get-caller-identity
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `aws sts get-caller-identity` | Returns the AWS account ID, IAM user/role ARN, and user ID for the active session — ensures the correct account is targeted before any irreversible action |

---

### Step 2: Locate the Target EC2 Instance

Resolved the instance ID for `datacenter-ec2` using its `Name` tag.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --region us-east-1 \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `describe-instances` | Lists EC2 instances matching the given filters |
| `--filters "Name=tag:Name,Values=datacenter-ec2"` | Filters by the `Name` tag to pinpoint the exact instance |
| `--region us-east-1` | Scopes the query to the required region |
| `--query "...[InstanceId,State.Name]"` | Extracts only the instance ID and current state from the full JSON response |
| `--output table` | Formats the result as a human-readable table |

**Expected Output:**

```text
------------------------------------
|         DescribeInstances        |
+----------------------+----------+
|  i-0123456789abcdef0 | running  |
+----------------------+----------+
```

---

### Step 3: Terminate the Instance

Issued the termination command using the resolved instance ID.

```bash
aws ec2 terminate-instances \
  --instance-ids i-0123456789abcdef0 \
  --region us-east-1
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `terminate-instances` | Permanently shuts down and deletes the specified EC2 instance — this action is irreversible |
| `--instance-ids` | The specific instance to terminate, resolved from the previous step |
| `--region us-east-1` | Ensures the action is applied in the correct region |

**Expected Output:**

```text
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-0123456789abcdef0",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

---

### Step 4: Wait for Termination to Complete

Blocked the CLI until the instance fully reached the `terminated` state before running the final verification.

```bash
aws ec2 wait instance-terminated \
  --instance-ids i-0123456789abcdef0 \
  --region us-east-1
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `wait instance-terminated` | Polls the instance state every 15 seconds (up to 40 attempts) and exits only once the instance reaches `terminated` — eliminates guesswork on timing |
| `--instance-ids` | The instance being polled |
| `--region us-east-1` | Confirms the correct regional scope for the poll |

> 💡 The `wait` command returns with no output on success. If it times out, re-run `describe-instances` manually to check the current state.

---

### Step 5: Confirm the Instance Is Terminated

```bash
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --region us-east-1 \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `describe-instances` | Re-queries the instance state for final confirmation |
| `--instance-ids` | Targets the specific instance that was just terminated |
| `--query "...[InstanceId,State.Name]"` | Returns only the ID and state for a clean, readable result |
| `--output table` | Formats output as a table for quick visual confirmation |

**Expected Final Output:**

```text
--------------------------------------
|         DescribeInstances          |
+----------------------+------------+
|  i-0123456789abcdef0 | terminated |
+----------------------+------------+
```

---

## ✅ Verification

### Confirm Terminated State

```bash
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --region us-east-1 \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table
```

**Expected Output:**

```text
--------------------------------------
|         DescribeInstances          |
+----------------------+------------+
|  i-0123456789abcdef0 | terminated |
+----------------------+------------+
```

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `aws sts get-caller-identity` | Verify authenticated AWS identity |
| `aws ec2 describe-instances --filters "Name=tag:Name,Values=<name>"` | Locate an instance by Name tag |
| `aws ec2 terminate-instances --instance-ids <id>` | Permanently terminate an EC2 instance |
| `aws ec2 wait instance-terminated --instance-ids <id>` | Block until the instance reaches terminated state |
| `aws ec2 describe-instances --instance-ids <id>` | Confirm final instance lifecycle state |

---

## 🏗️ Architecture / Workflow

```text
AWS CLI (aws-client host)
        │
        ▼
aws sts get-caller-identity
        │  (verify identity — irreversible action ahead)
        ▼
aws ec2 describe-instances
        │  (resolve instance ID from Name tag)
        ▼
aws ec2 terminate-instances
        │  (state: running → shutting-down)
        ▼
aws ec2 wait instance-terminated
        │  (polls every 15s until state changes)
        ▼
aws ec2 describe-instances
        │
        ▼
Instance State: terminated ✅
```

---

## 💡 Key Learnings

- Always verify CLI identity (`get-caller-identity`) before any destructive or irreversible operation.
- `terminate-instances` is permanent — once issued, an instance cannot be recovered unless an AMI or snapshot was taken beforehand.
- EC2 termination passes through an intermediate `shutting-down` state before reaching `terminated`; polling is required rather than assuming it's instant.
- `aws ec2 wait instance-terminated` is the cleanest way to block on lifecycle completion without writing a manual polling loop.
- Resolving instance IDs dynamically via `--filters` on `Name` tags keeps procedures portable and avoids hardcoded values.
- Terminated instances remain visible in `describe-instances` output for a short period before AWS removes them entirely.

---

## 🔒 Best Practices

- Always verify CLI identity before performing destructive operations, especially with temporary lab credentials.
- Resolve resource IDs dynamically via tags — never hardcode instance IDs into runbooks.
- Take an AMI or EBS snapshot before terminating any instance if there is any chance the data may be needed later.
- Explicitly scope all commands to `--region` to avoid accidentally targeting resources in the wrong region.
- Confirm the final `terminated` state before marking the task as complete — `shutting-down` is not the same as `terminated`.
- Regularly audit and decommission unused resources to minimize cost and reduce the cloud attack surface.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
