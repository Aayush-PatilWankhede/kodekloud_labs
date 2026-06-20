# 📅 Day 13 - Creating an AMI from an Existing EC2 Instance

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Cloud](https://img.shields.io/badge/Cloud-AWS-orange)
![Service](https://img.shields.io/badge/Service-EC2%20%7C%20AMI-yellow)
![Challenge](https://img.shields.io/badge/100--Days--AWS--DevOps--Challenge-Day%2013-blue)

---

## 📖 Challenge Overview

As part of the Nautilus DevOps team's phased migration strategy to AWS, large migration efforts are broken down into smaller, manageable tasks to reduce risk and ensure smoother execution. This challenge focuses on one such unit of work: creating an Amazon Machine Image (AMI) from an existing EC2 instance, enabling the team to capture a reusable, point-in-time snapshot of a server's configuration for future provisioning, backup, or migration purposes.

---

## 🎯 Objective

- Create an AMI from the existing EC2 instance `xfusion-ec2`.
- Name the AMI `xfusion-ec2-ami`.
- Ensure the AMI reaches the `available` state.
- Provision all resources strictly within the `us-east-1` region.

---

## 🛠️ Technologies / Services Used

| Category | Technology | Purpose |
|----------|------------|---------|
| Cloud    | AWS        | Cloud infrastructure provider |
| Service  | EC2        | Source compute instance |
| Service  | AMI (Amazon Machine Image) | Reusable image artifact for migration/backup |
| Tool     | AWS CLI    | Resource creation and verification |

---

## 📋 Challenge Requirements

- Source instance: `xfusion-ec2` (existing, running EC2 instance).
- Target AMI name: `xfusion-ec2-ami`.
- AMI must transition to and confirm an `available` state before completion.
- All actions must be scoped to the `us-east-1` region.
- Credentials provided via a lab-issued temporary IAM user (retrieved securely via the `showcreds` utility on the AWS client host — not embedded in this documentation).

---

## 🎯 Solution Approach

The task was completed entirely through the AWS CLI using a verify → locate → create → confirm workflow: first validating CLI authentication, then resolving the target instance's ID via its `Name` tag, issuing the `create-image` API call, and finally polling `describe-images` until the AMI's lifecycle state transitioned from `pending` to `available`.

---

## 🚀 Implementation Steps

### Step 1: Verify AWS CLI Authentication

Confirmed the CLI session was authenticated against the correct AWS account before making any changes.

```bash
aws sts get-caller-identity
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `aws sts get-caller-identity` | Returns the AWS account ID, IAM user/role ARN, and user ID of the currently authenticated session |

---

### Step 2: Locate the Target EC2 Instance

Resolved the instance ID for `xfusion-ec2` using its `Name` tag.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --region us-east-1 \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `describe-instances` | Lists EC2 instances matching the given criteria |
| `--filters "Name=tag:Name,Values=xfusion-ec2"` | Filters results to the instance tagged with `Name=xfusion-ec2` |
| `--region us-east-1` | Scopes the query to the required region |
| `--query "...[InstanceId,State.Name]"` | Extracts only the instance ID and current state from the response |
| `--output table` | Formats the result as a readable table |

**Expected Output:**

```text
------------------------------------
|         DescribeInstances        |
+----------------------+-----------+
|  i-0123456789abcdef0 |  running  |
+----------------------+-----------+
```

---

### Step 3: Create the AMI

Created the AMI from the identified instance.

```bash
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "xfusion-ec2-ami" \
  --region us-east-1
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `create-image` | Creates a new AMI from a running or stopped EC2 instance |
| `--instance-id` | Source instance the AMI is built from |
| `--name` | Name assigned to the resulting AMI |
| `--region us-east-1` | Ensures the AMI is created in the required region |

**Expected Output:**

```text
{
    "ImageId": "ami-0123456789abcdef0"
}
```

> 💡 By default, `create-image` reboots the instance briefly to ensure file system consistency. Add `--no-reboot` if a zero-downtime image is required instead (with a small trade-off in guaranteed consistency).

---

### Step 4: Verify the AMI Reaches an Available State

Polled the AMI status until it transitioned from `pending` to `available`.

```bash
aws ec2 describe-images \
  --image-ids ami-0123456789abcdef0 \
  --region us-east-1 \
  --query "Images[*].[ImageId,Name,State]" \
  --output table
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `describe-images` | Retrieves metadata about one or more AMIs |
| `--image-ids` | The specific AMI to check |
| `--query "...[ImageId,Name,State]"` | Returns only the image ID, name, and lifecycle state |
| `--output table` | Formats the result as a readable table |

**Expected Final Output:**

```text
-------------------------------------------------
|                 DescribeImages                 |
+----------------------+-------------------+-----------+
|  ami-0123456789abcdef0 |  xfusion-ec2-ami |  available |
+----------------------+-------------------+-----------+
```

---

## ✅ Verification

### Continuous Status Check (Optional)

For convenience, the AMI status can be polled automatically instead of re-running the command manually.

```bash
watch -n 5 "aws ec2 describe-images --image-ids ami-0123456789abcdef0 --region us-east-1 --query 'Images[*].State' --output table"
```

**Command Explanation**

| Option | Description |
|--------|-------------|
| `watch` | Repeatedly executes a command at a fixed interval |
| `-n 5` | Sets the refresh interval to 5 seconds |
| `"..."` | The full AWS CLI command, quoted so `watch` treats it as a single unit |

Exit the watch loop with:

```bash
CTRL + C
```

**Expected Output:**

```text
available
```

---

## 📑 Command Reference

| Command | Purpose |
|---------|---------|
| `aws sts get-caller-identity` | Verify authenticated AWS identity |
| `aws ec2 describe-instances --filters "Name=tag:Name,Values=<name>"` | Find an instance by its Name tag |
| `aws ec2 create-image --instance-id <id> --name <name>` | Create an AMI from an instance |
| `aws ec2 describe-images --image-ids <id>` | Check AMI metadata and lifecycle state |
| `watch -n 5 "<command>"` | Auto-refresh a command's output at intervals |

---

## 🏗️ Architecture / Workflow

```text
AWS CLI (aws-client host)
        │
        ▼
 aws sts get-caller-identity
        │  (verify identity)
        ▼
 aws ec2 describe-instances
        │  (resolve instance ID from Name tag)
        ▼
 aws ec2 create-image
        │  (AMI state: pending)
        ▼
 aws ec2 describe-images (poll)
        │
        ▼
 AMI State: available ✅
```

---

## 💡 Key Learnings

- AMIs pass through a `pending` state before becoming `available`; status must be polled rather than assumed instantaneous.
- `describe-instances` filtering by `tag:Name` is a reliable way to resolve resource IDs without hardcoding them.
- `create-image` reboots the source instance by default to ensure a consistent file system snapshot — `--no-reboot` exists for cases where availability takes priority over guaranteed consistency.
- `--query` with JMESPath syntax is an efficient way to extract only relevant fields from verbose AWS CLI JSON responses.
- `watch` is useful for monitoring asynchronous AWS operations without manually re-running commands.

---

## 🔒 Best Practices

- Always verify the active AWS CLI identity (`get-caller-identity`) before performing resource operations, especially with temporary/lab credentials.
- Resolve resource IDs dynamically via tags rather than hardcoding them, to keep scripts portable across environments.
- Explicitly pass `--region` on every command rather than relying on default CLI configuration, to avoid accidental cross-region resource creation.
- Confirm a resource has reached its final lifecycle state (`available`) before considering a task complete.
- Never store AWS console credentials, access keys, or passwords in documentation or version control.

---

## 🏁 Challenge Status

✅ **Successfully Completed**

---

<p align="center">
  <i>Part of the <a href="#">100 Days AWS DevOps Challenge</a></i>
</p>
