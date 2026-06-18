# Day 11 - Attach an Elastic Network Interface (ENI) to an Amazon EC2 Instance

## Overview

This challenge focuses on attaching an existing Elastic Network Interface (ENI) to an existing Amazon EC2 instance. Elastic Network Interfaces provide additional networking capabilities and can be attached or detached from EC2 instances as required.

The objective is to attach the ENI named `devops-eni` to the EC2 instance named `devops-ec2` and verify that the attachment status is successfully updated.

---

## Objective

- Verify AWS account access
- Locate the target EC2 instance
- Locate the existing Elastic Network Interface
- Attach the ENI to the EC2 instance
- Verify the ENI attachment status
- Confirm the EC2 instance has completed initialization checks

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| Elastic Network Interface (ENI) | Provide additional network interfaces |
| AWS CLI | Manage AWS resources |
| AWS STS | Verify AWS identity |

---

## Implementation

### Verify AWS Identity

Verify that the AWS CLI is authenticated with the correct AWS account.

```bash
aws sts get-caller-identity
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `aws sts get-caller-identity` | Displays the current AWS account and IAM identity |

Example Output:

```json
{
  "UserId": "XXXXXXXXXXXX",
  "Account": "XXXXXXXXXXXX",
  "Arn": "arn:aws:iam::XXXXXXXXXXXX:user/username"
}
```

---

### Locate the EC2 Instance

Retrieve the instance ID of the EC2 instance named `devops-ec2`.

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
| `describe-instances` | Retrieves EC2 instance details |
| `--filters` | Searches resources using tags |
| `Name=tag:Name` | Filters by Name tag |
| `Values=devops-ec2` | Finds the target EC2 instance |
| `--query` | Returns only selected fields |
| `--output table` | Displays output in table format |

Example Output:

```text
------------------------------------------------
|              DescribeInstances               |
+----------------------+-----------------------+
| i-0123456789abcdef0  | running               |
+----------------------+-----------------------+
```

---

### Locate the Elastic Network Interface

Retrieve the ENI ID of the network interface named `devops-eni`.

```bash
aws ec2 describe-network-interfaces \
    --filters "Name=tag:Name,Values=devops-eni" \
    --region us-east-1 \
    --query 'NetworkInterfaces[*].[NetworkInterfaceId,Status]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-network-interfaces` | Retrieves ENI details |
| `--filters` | Searches network interfaces using tags |
| `Name=tag:Name` | Filters by Name tag |
| `Values=devops-eni` | Finds the target ENI |
| `Status` | Displays ENI state |

Example Output:

```text
-----------------------------------------------------
|          DescribeNetworkInterfaces                |
+------------------------+--------------------------+
| eni-0abc123456789def0  | available                |
+------------------------+--------------------------+
```

The ENI status should be:

```text
available
```

which indicates that the interface is ready to be attached.

---

### Attach the ENI to the EC2 Instance

Attach the Elastic Network Interface as a secondary network interface.

```bash
aws ec2 attach-network-interface \
    --network-interface-id eni-0abc123456789def0 \
    --instance-id i-0123456789abcdef0 \
    --device-index 1 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `attach-network-interface` | Attaches an ENI to an EC2 instance |
| `--network-interface-id` | Specifies the ENI to attach |
| `--instance-id` | Specifies the target EC2 instance |
| `--device-index 1` | Attaches the ENI as a secondary network interface |
| `--region us-east-1` | Executes command in the specified AWS region |

Example Output:

```json
{
  "AttachmentId": "eni-attach-0123456789abcdef0"
}
```

---

## Verification

### Verify ENI Attachment Status

Confirm that the ENI is attached successfully.

```bash
aws ec2 describe-network-interfaces \
    --network-interface-ids eni-0abc123456789def0 \
    --region us-east-1 \
    --query 'NetworkInterfaces[*].[NetworkInterfaceId,Status,Attachment.Status,Attachment.InstanceId]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `Status` | Displays ENI state |
| `Attachment.Status` | Displays attachment status |
| `Attachment.InstanceId` | Displays attached EC2 instance |

Expected Output:

```text
-----------------------------------------------------------------------------------
|                  DescribeNetworkInterfaces                                      |
+------------------------+---------+-----------+----------------------+
| eni-0abc123456789def0  | in-use  | attached  | i-0123456789abcdef0 |
+------------------------+---------+-----------+----------------------+
```

Important Values:

```text
Status            = in-use
Attachment.Status = attached
```

These values confirm that the ENI is successfully attached.

---

### Verify EC2 Instance Status Checks

Ensure that the instance initialization and health checks have completed successfully.

```bash
aws ec2 describe-instance-status \
    --instance-ids i-0123456789abcdef0 \
    --region us-east-1 \
    --query 'InstanceStatuses[*].[InstanceId,InstanceState.Name,SystemStatus.Status,InstanceStatus.Status]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instance-status` | Retrieves EC2 health status |
| `SystemStatus.Status` | AWS infrastructure health check |
| `InstanceStatus.Status` | EC2 instance health check |

Expected Output:

```text
-----------------------------------------------------------------------
|                    DescribeInstanceStatus                           |
+----------------------+----------+------+------+
| i-0123456789abcdef0  | running  | ok   | ok   |
+----------------------+----------+------+------+
```

Important Values:

```text
SystemStatus.Status   = ok
InstanceStatus.Status = ok
```

These values confirm that the instance has completed initialization and is healthy.

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify AWS Identity │
│ AWS STS             │
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
│ Locate ENI          │
│ devops-eni          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Attach ENI          │
│ Device Index = 1    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Attachment   │
│ Status = attached   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Health       │
│ Status Checks = ok  │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws sts get-caller-identity` | Verify AWS account identity |
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 describe-network-interfaces` | Retrieve ENI details |
| `aws ec2 attach-network-interface` | Attach ENI to an EC2 instance |
| `aws ec2 describe-instance-status` | Verify instance health checks |

---

## Key Learnings

- Learned how Elastic Network Interfaces are managed in AWS.
- Retrieved ENI details using AWS CLI.
- Attached a secondary network interface to an EC2 instance.
- Verified ENI attachment status and usage state.
- Checked EC2 instance initialization and health status.

---

## Best Practices

- Verify instance health checks before modifying resources.
- Attach secondary ENIs using an appropriate device index.
- Use resource tags for easier identification and automation.
- Validate ENI attachment status after configuration changes.
- Monitor instance networking after attaching additional interfaces.

---

## Challenge Status

✅ Successfully Completed
