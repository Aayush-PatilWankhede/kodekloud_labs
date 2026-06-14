# Day 09 - Enable EC2 Termination Protection Using AWS CLI

## Overview

This challenge focuses on securing an existing Amazon EC2 instance by enabling **Termination Protection**. This feature prevents accidental deletion of critical EC2 instances through the AWS Console, AWS CLI, or AWS APIs.

The objective is to locate the existing EC2 instance named `devops-ec2`, enable termination protection, and verify that the protection has been successfully configured.

---

## Objective

- Locate the EC2 instance named `devops-ec2`
- Enable EC2 Termination Protection
- Verify that termination protection is active
- Prevent accidental deletion of the instance

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| AWS CLI | Interact with AWS services |
| EC2 Instance Attributes | Configure instance settings |
| EC2 Termination Protection | Prevent accidental instance deletion |

---

## Implementation

### Locate the EC2 Instance

Retrieve the instance ID and current state of the target EC2 instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=devops-ec2" \
    --region us-east-1 \
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instances` | Retrieves EC2 instance information |
| `--filters` | Filters resources based on specified criteria |
| `Name=tag:Name` | Searches using the Name tag |
| `Values=devops-ec2` | Identifies the target instance |
| `--region us-east-1` | Executes command in the specified AWS region |
| `--query` | Extracts specific fields from the response |
| `InstanceId` | Unique identifier of the EC2 instance |
| `State.Name` | Displays the current instance state |
| `--output table` | Formats output in a readable table |

Example Output:

```text
------------------------------------------------
|              DescribeInstances               |
+----------------------+-----------------------+
| i-0123456789abcdef0  | running               |
+----------------------+-----------------------+
```

---

### Enable Termination Protection

Use the instance ID obtained from the previous step to enable termination protection.

```bash
aws ec2 modify-instance-attribute \
    --instance-id INSTANCE_ID \
    --disable-api-termination \
    --region us-east-1
```

Example:

```bash
aws ec2 modify-instance-attribute \
    --instance-id i-0123456789abcdef0 \
    --disable-api-termination \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `modify-instance-attribute` | Modifies EC2 instance configuration |
| `--instance-id` | Specifies the target EC2 instance |
| `--disable-api-termination` | Enables termination protection |
| `--region us-east-1` | Executes command in the specified AWS region |

Once enabled, the instance cannot be terminated through:

- AWS Management Console
- AWS CLI
- AWS SDKs
- AWS APIs

unless termination protection is explicitly disabled first.

---

## Verification

### Verify Termination Protection Status

Confirm that termination protection has been successfully enabled.

```bash
aws ec2 describe-instance-attribute \
    --instance-id INSTANCE_ID \
    --attribute disableApiTermination \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instance-attribute` | Retrieves a specific EC2 attribute |
| `--instance-id` | Specifies the target instance |
| `--attribute disableApiTermination` | Checks termination protection status |
| `--region us-east-1` | Executes command in the specified AWS region |

Expected Output:

```json
{
  "DisableApiTermination": {
    "Value": true
  },
  "InstanceId": "i-0123456789abcdef0"
}
```

The important value is:

```json
{
  "Value": true
}
```

which confirms that termination protection is enabled.

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
│ Retrieve            │
│ Instance ID         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Enable Termination  │
│ Protection          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify              │
│ disableApiTermination│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Instance Protected  │
│ From Accidental     │
│ Deletion            │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 modify-instance-attribute` | Modify EC2 instance settings |
| `aws ec2 describe-instance-attribute` | Verify EC2 instance attributes |
| `--disable-api-termination` | Enable termination protection |

---

## Key Learnings

- Learned how to identify EC2 instances using resource tags.
- Configured EC2 instance attributes through AWS CLI.
- Enabled termination protection for critical infrastructure.
- Verified EC2 protection settings using instance attributes.
- Improved infrastructure safety and operational resilience.

---

## Best Practices

- Enable termination protection on production and critical workloads.
- Use resource tags for easier infrastructure management.
- Verify protection settings after configuration changes.
- Follow least-privilege IAM principles for EC2 administration.
- Regularly audit EC2 configurations to prevent accidental resource deletion.

---

## Challenge Status

✅ Successfully Completed
