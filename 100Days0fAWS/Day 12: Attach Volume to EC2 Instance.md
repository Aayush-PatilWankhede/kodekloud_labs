# Day 12 - Attach an EBS Volume to an EC2 Instance Using AWS CLI

## Overview

As part of the AWS migration process, the Nautilus DevOps team needs to attach an existing EBS volume to an existing EC2 instance. This enables additional storage to be made available to the instance for application data, backups, logs, or other workloads.

---

## Objective

- Locate the EC2 instance named `nautilus-ec2`
- Locate the EBS volume named `nautilus-volume`
- Verify both resources are in the same Availability Zone
- Attach the volume to the instance
- Use the device name `/dev/sdb`
- Verify the volume attachment status

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Manage virtual servers |
| Amazon EBS | Persistent block storage |
| AWS CLI | AWS resource management |
| AWS STS | Verify AWS identity |

---

## Implementation

### Verify AWS Identity

Before making changes, verify the active AWS account and credentials.

```bash
aws sts get-caller-identity
```

#### Command Explanation

| Command | Description |
|----------|-------------|
| `aws sts get-caller-identity` | Displays the currently authenticated AWS account and IAM identity |

---

### Find the EC2 Instance ID

Retrieve the instance ID and Availability Zone of the EC2 instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=nautilus-ec2" \
    --region us-east-1 \
    --query "Reservations[*].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone]" \
    --output table
```

#### Expected Output

```text
---------------------------------------------------
|               DescribeInstances                 |
+----------------------+----------+--------------+
| i-0123456789abcdef0  | running  | us-east-1a   |
+----------------------+----------+--------------+
```

#### Command Explanation

| Parameter | Description |
|------------|-------------|
| `describe-instances` | Retrieves EC2 instance details |
| `--filters` | Searches instances using tags |
| `Name=tag:Name` | Filters by Name tag |
| `Values=nautilus-ec2` | Finds the target instance |
| `Placement.AvailabilityZone` | Shows instance Availability Zone |
| `--output table` | Formats output as a table |

---

### Find the Volume ID

Retrieve the EBS volume ID and Availability Zone.

```bash
aws ec2 describe-volumes \
    --filters "Name=tag:Name,Values=nautilus-volume" \
    --region us-east-1 \
    --query "Volumes[*].[VolumeId,State,AvailabilityZone]" \
    --output table
```

#### Expected Output

```text
---------------------------------------------------
|                DescribeVolumes                  |
+----------------------+-----------+-------------+
| vol-0abc123def45678  | available | us-east-1a  |
+----------------------+-----------+-------------+
```

#### Command Explanation

| Parameter | Description |
|------------|-------------|
| `describe-volumes` | Retrieves EBS volume details |
| `VolumeId` | Unique EBS volume identifier |
| `State` | Current volume state |
| `AvailabilityZone` | Volume Availability Zone |

> **Important:** The EC2 instance and EBS volume must be in the same Availability Zone before attachment.

---

### Attach the Volume

Replace the placeholders with the actual IDs obtained from previous steps.

```bash
aws ec2 attach-volume \
    --volume-id vol-xxxxxxxxxxxxxxxxx \
    --instance-id i-xxxxxxxxxxxxxxxxx \
    --device /dev/sdb \
    --region us-east-1
```

#### Command Explanation

| Parameter | Description |
|------------|-------------|
| `attach-volume` | Attaches an EBS volume to an EC2 instance |
| `--volume-id` | Specifies the EBS volume |
| `--instance-id` | Specifies the target EC2 instance |
| `--device /dev/sdb` | Assigns the device name required by the task |
| `--region us-east-1` | Specifies the AWS region |

#### Expected Output

```json
{
  "AttachTime": "2026-05-22T05:00:00+00:00",
  "Device": "/dev/sdb",
  "InstanceId": "i-xxxxxxxxxxxxxxxxx",
  "State": "attaching",
  "VolumeId": "vol-xxxxxxxxxxxxxxxxx"
}
```

---

## Verification

### Verify Volume Attachment

Check that the volume is successfully attached to the EC2 instance.

```bash
aws ec2 describe-volumes \
    --volume-ids vol-xxxxxxxxxxxxxxxxx \
    --region us-east-1 \
    --query "Volumes[*].Attachments[*].[InstanceId,Device,State]" \
    --output table
```

#### Expected Output

```text
---------------------------------------------------
|                DescribeVolumes                  |
+----------------------+-----------+-------------+
| i-xxxxxxxxxxxxxxxxx  | /dev/sdb  | attached    |
+----------------------+-----------+-------------+
```

---

## Workflow

```text
┌─────────────────────┐
│ Verify AWS Identity │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Find EC2 Instance   │
│ nautilus-ec2        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Find EBS Volume     │
│ nautilus-volume     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Both Are In  │
│ Same Availability   │
│ Zone                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Attach Volume       │
│ Device: /dev/sdb    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Attachment   │
│ State = attached    │
└─────────────────────┘
```

---

## Command Reference

| Command | Purpose |
|----------|----------|
| `aws sts get-caller-identity` | Verify active AWS identity |
| `aws ec2 describe-instances` | Retrieve EC2 instance details |
| `aws ec2 describe-volumes` | Retrieve EBS volume details |
| `aws ec2 attach-volume` | Attach EBS volume to EC2 instance |
| `aws ec2 describe-volumes --query Attachments` | Verify volume attachment |

---

## Key Learnings

- Learned how to identify AWS resources using tags.
- Understood the relationship between EC2 instances and EBS volumes.
- Learned that EBS volumes can only be attached within the same Availability Zone.
- Practiced attaching additional storage to EC2 instances using AWS CLI.
- Verified volume attachment using AWS resource queries.

---

## Best Practices

- Always verify the Availability Zone before attaching an EBS volume.
- Use descriptive Name tags for easier resource identification.
- Confirm volume attachment status before proceeding with application deployment.
- Use AWS CLI queries to retrieve only required information.
- Validate storage resources after every infrastructure change.

---

## Final Validation Checklist

| Validation Item | Status |
|-----------------|---------|
| EC2 Instance Located | ✅ |
| EBS Volume Located | ✅ |
| Same Availability Zone Verified | ✅ |
| Device Name Set to `/dev/sdb` | ✅ |
| Volume Successfully Attached | ✅ |
| Attachment Status Verified | ✅ |

---

## Challenge Status

✅ Successfully Completed
