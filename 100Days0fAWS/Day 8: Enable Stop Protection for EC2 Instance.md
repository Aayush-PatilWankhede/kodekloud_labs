# Day 08 - Enable Stop Protection for an Amazon EC2 Instance

## Overview

This challenge focuses on enhancing the safety and availability of an existing EC2 instance by enabling **Stop Protection**. Stop Protection prevents an instance from being accidentally stopped through the AWS Management Console, AWS CLI, or AWS APIs.

The objective is to locate the EC2 instance named `nautilus-ec2`, enable stop protection, and verify that the protection has been successfully applied.

---

## Objective

- Locate the EC2 instance named `nautilus-ec2`
- Enable Stop Protection for the instance
- Verify that Stop Protection is enabled
- Prevent accidental instance shutdowns

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| AWS CLI | Interact with AWS services |
| EC2 Instance Attributes | Configure instance settings |
| EC2 Stop Protection | Prevent accidental instance stops |

---

## Implementation

### Locate the EC2 Instance

Retrieve the instance ID and current state of the target EC2 instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=nautilus-ec2" \
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
| `Values=nautilus-ec2` | Identifies the target instance |
| `--region us-east-1` | Executes command in the specified region |
| `--query` | Returns only selected attributes |
| `InstanceId` | Unique identifier of the EC2 instance |
| `State.Name` | Displays the current instance state |
| `--output table` | Formats output as a table |

Example Output:

```text
------------------------------------------------
|              DescribeInstances               |
+----------------------+-----------------------+
| i-0123456789abcdef0  | running               |
+----------------------+-----------------------+
```

---

### Enable Stop Protection

Use the instance ID obtained from the previous step to enable Stop Protection.

```bash
aws ec2 modify-instance-attribute \
    --instance-id INSTANCE_ID \
    --disable-api-stop '{"Value":true}' \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `modify-instance-attribute` | Modifies EC2 instance settings |
| `--instance-id` | Specifies the target instance |
| `--disable-api-stop` | Enables or disables Stop Protection |
| `{"Value":true}` | Enables Stop Protection |
| `--region us-east-1` | Executes command in the specified region |

When enabled, the instance cannot be stopped accidentally through:

- AWS Management Console
- AWS CLI
- AWS SDKs
- AWS APIs

---

## Verification

### Verify Stop Protection Status

Confirm that Stop Protection has been enabled successfully.

```bash
aws ec2 describe-instance-attribute \
    --instance-id INSTANCE_ID \
    --attribute disableApiStop \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instance-attribute` | Retrieves a specific EC2 attribute |
| `--instance-id` | Specifies the target instance |
| `--attribute disableApiStop` | Checks Stop Protection status |
| `--region us-east-1` | Executes command in the specified region |

Expected Output:

```json
{
  "DisableApiStop": {
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

which confirms that Stop Protection is enabled.

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
│ nautilus-ec2        │
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
│ Enable Stop         │
│ Protection          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify              │
│ disableApiStop      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Instance Protected  │
│ From Accidental     │
│ Stops               │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 modify-instance-attribute` | Modify EC2 instance settings |
| `aws ec2 describe-instance-attribute` | Verify instance attributes |
| `--disable-api-stop '{"Value":true}'` | Enable Stop Protection |

---

## Key Learnings

- Learned how to identify EC2 instances using tags.
- Configured EC2 instance attributes using AWS CLI.
- Enabled Stop Protection to prevent accidental shutdowns.
- Verified EC2 instance protection settings.
- Improved infrastructure reliability through operational safeguards.

---

## Best Practices

- Enable Stop Protection for critical production instances.
- Verify instance attributes after making configuration changes.
- Use resource tags for easier instance identification.
- Restrict administrative access using IAM least-privilege principles.
- Regularly audit EC2 configurations to prevent accidental disruptions.

---

## Challenge Status

✅ Successfully Completed
