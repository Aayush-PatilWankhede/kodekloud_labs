# Day 07 - Modify an Amazon EC2 Instance Type Using AWS CLI

## Overview

This challenge focuses on optimizing AWS resource utilization by modifying the instance type of an existing EC2 instance. As part of infrastructure right-sizing, the instance type of `devops-ec2` must be changed from `t2.micro` to `t2.nano`.

Since instance type modifications require the instance to be in a stopped state, it is important to first verify that all EC2 status checks have completed successfully before making any changes.

---

## Objective

- Identify the EC2 instance named `devops-ec2`
- Verify that EC2 status checks have completed successfully
- Stop the EC2 instance safely
- Change the instance type from `t2.micro` to `t2.nano`
- Restart the instance
- Verify that the instance is running with the new instance type

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| AWS CLI | Interact with AWS services |
| EC2 Instance Status Checks | Verify instance health |
| EC2 Instance Attributes | Modify instance configuration |

---

## Implementation

### Locate the EC2 Instance

Retrieve the instance ID, current instance type, and running state of the target instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=devops-ec2" \
    --region us-east-1 \
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instances` | Retrieves EC2 instance details |
| `--filters` | Filters instances by tag |
| `Name=tag:Name,Values=devops-ec2` | Searches for the target instance |
| `--query` | Returns selected attributes only |
| `--output table` | Displays results in table format |

Example Output:

```text
---------------------------------------------------
|                DescribeInstances                |
+----------------------+-------------+-----------+
| i-0123456789abcdef0  | t2.micro    | running   |
+----------------------+-------------+-----------+
```

---

### Verify EC2 Status Checks

Ensure the instance has completed initialization and passed all health checks.

```bash
aws ec2 describe-instance-status \
    --instance-ids <INSTANCE-ID> \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instance-status` | Retrieves EC2 health status |
| `--instance-ids` | Specifies the target instance |
| `--region us-east-1` | Executes command in the required region |

Verify the following values:

```text
InstanceStatus.Status = ok
SystemStatus.Status   = ok
```

These values indicate that the instance has successfully completed all status checks.

---

### Stop the EC2 Instance

The instance must be stopped before modifying its instance type.

```bash
aws ec2 stop-instances \
    --instance-ids <INSTANCE-ID> \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `stop-instances` | Gracefully stops the EC2 instance |
| `--instance-ids` | Specifies the target instance |

---

### Wait for the Instance to Stop

Ensure the instance reaches the fully stopped state before proceeding.

```bash
aws ec2 wait instance-stopped \
    --instance-ids <INSTANCE-ID> \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `wait instance-stopped` | Waits until the instance is fully stopped |
| `--instance-ids` | Specifies the target instance |

This prevents modification attempts while the instance is still shutting down.

---

### Modify the Instance Type

Change the EC2 instance type from `t2.micro` to `t2.nano`.

```bash
aws ec2 modify-instance-attribute \
    --instance-id <INSTANCE-ID> \
    --instance-type "{\"Value\": \"t2.nano\"}" \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `modify-instance-attribute` | Updates EC2 instance attributes |
| `--instance-id` | Specifies the target instance |
| `--instance-type` | Defines the new instance type |
| `t2.nano` | New instance size selected for optimization |

---

### Start the EC2 Instance

Restart the instance after modifying its configuration.

```bash
aws ec2 start-instances \
    --instance-ids <INSTANCE-ID> \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `start-instances` | Starts a stopped EC2 instance |
| `--instance-ids` | Specifies the target instance |

---

## Verification

### Verify Instance Type and Running State

Confirm that the instance is running and using the updated instance type.

```bash
aws ec2 describe-instances \
    --instance-ids <INSTANCE-ID> \
    --region us-east-1 \
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instances` | Retrieves EC2 instance details |
| `--query` | Filters the output to relevant attributes |
| `--output table` | Displays results in tabular format |

Expected Output:

```text
---------------------------------------------------
|                DescribeInstances                |
+----------------------+-------------+-----------+
| i-0123456789abcdef0  | t2.nano     | running   |
+----------------------+-------------+-----------+
```

The important values are:

```text
Instance Type : t2.nano
State         : running
```

which confirm that the modification was completed successfully.

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Locate EC2 Instance │
│ devops-ec2          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Status       │
│ Checks              │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Stop EC2 Instance   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Wait Until Stopped  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Change Instance     │
│ Type to t2.nano     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Start EC2 Instance  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Running      │
│ State & Type        │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 describe-instance-status` | Check instance health status |
| `aws ec2 stop-instances` | Stop an EC2 instance |
| `aws ec2 wait instance-stopped` | Wait for instance shutdown |
| `aws ec2 modify-instance-attribute` | Change EC2 instance type |
| `aws ec2 start-instances` | Start an EC2 instance |

---

## Key Learnings

- Learned how to identify EC2 instances using tags.
- Verified EC2 status checks before performing modifications.
- Understood the process of changing EC2 instance types.
- Used AWS CLI waiters to automate state transitions.
- Verified infrastructure changes using AWS CLI queries.

---

## Best Practices

- Always verify EC2 status checks before performing maintenance.
- Stop instances gracefully before modifying instance attributes.
- Use AWS CLI wait commands to avoid timing issues.
- Validate configuration changes after restarting instances.
- Right-size EC2 instances regularly to optimize cloud costs.

---

## Challenge Status

✅ Successfully Completed
