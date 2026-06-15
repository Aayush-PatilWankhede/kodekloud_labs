# Day 10 - Associate an Elastic IP with an Amazon EC2 Instance

## Overview

This challenge focuses on assigning a pre-existing **Elastic IP (EIP)** to an existing Amazon EC2 instance. Elastic IPs provide static public IPv4 addresses that remain associated with your AWS account and can be reassigned between instances when needed.

The objective is to attach the Elastic IP named `devops-ec2-eip` to the EC2 instance named `devops-ec2` in the `us-east-1` region.

---

## Objective

- Identify the EC2 instance named `devops-ec2`
- Retrieve the Elastic IP allocation details
- Associate the Elastic IP with the target EC2 instance
- Verify successful Elastic IP attachment

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| Elastic IP | Static public IPv4 address |
| AWS CLI | Interact with AWS services |
| EC2 Networking | Associate public IP addresses |

---

## Implementation

### Locate the EC2 Instance

Retrieve the instance ID of the target EC2 instance.

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
| `--filters` | Filters resources based on tags |
| `Name=tag:Name` | Searches using the Name tag |
| `Values=devops-ec2` | Identifies the target instance |
| `--region us-east-1` | Executes command in the specified AWS region |
| `--query` | Returns only selected attributes |
| `InstanceId` | Unique EC2 instance identifier |
| `State.Name` | Displays current instance state |
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

### Retrieve Elastic IP Details

Obtain the Allocation ID of the Elastic IP named `devops-ec2-eip`.

```bash
aws ec2 describe-addresses \
    --filters "Name=tag:Name,Values=devops-ec2-eip" \
    --region us-east-1 \
    --query 'Addresses[*].[PublicIp,AllocationId,AssociationId]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-addresses` | Retrieves Elastic IP information |
| `--filters` | Filters Elastic IP resources |
| `Name=tag:Name` | Searches using the Name tag |
| `Values=devops-ec2-eip` | Identifies the target Elastic IP |
| `AllocationId` | Unique identifier of the Elastic IP |
| `AssociationId` | Existing association details if attached |
| `PublicIp` | Public IPv4 address assigned to the Elastic IP |

Example Output:

```text
----------------------------------------------------------
|                 DescribeAddresses                      |
+---------------+------------------------+--------------+
| 54.xx.xx.xx   | eipalloc-0abc123456789 | None         |
+---------------+------------------------+--------------+
```

---

### Associate the Elastic IP with the EC2 Instance

Attach the Elastic IP to the target EC2 instance.

```bash
aws ec2 associate-address \
    --instance-id INSTANCE_ID \
    --allocation-id ALLOCATION_ID \
    --region us-east-1
```

Example:

```bash
aws ec2 associate-address \
    --instance-id i-0123456789abcdef0 \
    --allocation-id eipalloc-0abc123456789 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `associate-address` | Associates an Elastic IP with an EC2 instance |
| `--instance-id` | Specifies the target EC2 instance |
| `--allocation-id` | Specifies the Elastic IP allocation |
| `--region us-east-1` | Executes command in the specified AWS region |

Once completed, the Elastic IP becomes the public IP address of the instance.

---

## Verification

### Verify Elastic IP Association

Confirm that the Elastic IP is attached to the correct EC2 instance.

```bash
aws ec2 describe-addresses \
    --filters "Name=tag:Name,Values=devops-ec2-eip" \
    --region us-east-1 \
    --query 'Addresses[*].[PublicIp,InstanceId,AssociationId]' \
    --output table
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-addresses` | Retrieves Elastic IP information |
| `InstanceId` | Displays the associated EC2 instance |
| `AssociationId` | Displays Elastic IP attachment information |

Expected Output:

```text
-----------------------------------------------------------------
|                    DescribeAddresses                          |
+---------------+----------------------+-----------------------+
| 54.xx.xx.xx   | i-0123456789abcdef0  | eipassoc-xxxxxxxxxxxx |
+---------------+----------------------+-----------------------+
```

The important field is:

```text
InstanceId = i-0123456789abcdef0
```

which confirms that the Elastic IP is successfully attached to the EC2 instance.

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
│ Retrieve Elastic IP │
│ Allocation ID       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Associate Elastic   │
│ IP with Instance    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Association  │
│ InstanceId Present  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Elastic IP Attached │
│ Successfully        │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 describe-addresses` | Retrieve Elastic IP details |
| `aws ec2 associate-address` | Attach an Elastic IP to an EC2 instance |
| `AllocationId` | Unique Elastic IP identifier |
| `InstanceId` | Unique EC2 instance identifier |

---

## Key Learnings

- Learned how Elastic IPs are managed in AWS.
- Retrieved Elastic IP allocation details using AWS CLI.
- Associated an Elastic IP with an EC2 instance.
- Verified Elastic IP assignments through AWS CLI.
- Understood the role of static public IP addresses in cloud environments.

---

## Best Practices

- Use Elastic IPs only when a static public address is required.
- Release unused Elastic IPs to avoid unnecessary charges.
- Tag Elastic IP resources for easier identification.
- Verify associations after networking changes.
- Document public IP assignments for operational visibility.

---

## Challenge Status

✅ Successfully Completed
