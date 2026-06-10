# Day 05 - Create an AWS EBS GP3 Volume

## Overview

This challenge focuses on creating an Amazon Elastic Block Store (EBS) volume using the AWS CLI. EBS volumes provide persistent block-level storage for Amazon EC2 instances and are commonly used for operating systems, databases, and application storage.

The objective is to create a `gp3` EBS volume named `datacenter-volume` with a size of `2 GiB` in the `us-east-1` region and verify its configuration.

---

## Objective

- Create an EBS volume named `datacenter-volume`
- Configure the volume type as `gp3`
- Allocate `2 GiB` of storage
- Create the volume in the `us-east-1` region
- Verify the volume configuration and status

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EBS | Provides persistent block storage |
| Amazon EC2 | Manages EBS volumes |
| AWS CLI | Interact with AWS services |
| AWS Tags | Organize and identify resources |

---

## Implementation

### Create the EBS Volume

Create a new General Purpose SSD (`gp3`) EBS volume with a size of `2 GiB`.

```bash
aws ec2 create-volume \
    --availability-zone us-east-1a \
    --size 2 \
    --volume-type gp3 \
    --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=datacenter-volume}]' \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `create-volume` | Creates a new EBS volume |
| `--availability-zone us-east-1a` | Specifies the Availability Zone where the volume will be created |
| `--size 2` | Allocates 2 GiB of storage |
| `--volume-type gp3` | Creates a General Purpose SSD (gp3) volume |
| `--tag-specifications` | Assigns tags during resource creation |
| `Key=Name,Value=datacenter-volume` | Sets the Name tag for easier identification |
| `--region us-east-1` | Creates the volume in the required AWS region |

Expected Output:

```json
{
  "AvailabilityZone": "us-east-1a",
  "Size": 2,
  "VolumeType": "gp3",
  "State": "creating",
  "VolumeId": "vol-xxxxxxxxxxxxxxxxx"
}
```

---

## Verification

### Verify Volume Creation

Retrieve the volume details using the assigned Name tag.

```bash
aws ec2 describe-volumes \
    --filters "Name=tag:Name,Values=datacenter-volume" \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-volumes` | Retrieves information about EBS volumes |
| `--filters` | Filters the results based on specified criteria |
| `Name=tag:Name,Values=datacenter-volume` | Searches for the volume using its Name tag |
| `--region us-east-1` | Executes the command in the specified AWS region |

Expected Output:

```json
{
  "Volumes": [
    {
      "VolumeId": "vol-xxxxxxxxxxxxxxxxx",
      "Size": 2,
      "VolumeType": "gp3",
      "State": "available",
      "AvailabilityZone": "us-east-1a"
    }
  ]
}
```

The important fields are:

```json
{
  "VolumeType": "gp3",
  "Size": 2,
  "State": "available",
  "AvailabilityZone": "us-east-1a"
}
```

which confirm that the volume has been created successfully with the required configuration.

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create EBS Volume   │
│ datacenter-volume   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Configure Type      │
│ gp3 SSD             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Allocate Storage    │
│ 2 GiB               │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Apply Name Tag      │
│ datacenter-volume   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Volume       │
│ Configuration       │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 create-volume` | Create a new EBS volume |
| `aws ec2 describe-volumes` | Retrieve and verify EBS volume details |

---

## Key Learnings

- Learned how to create EBS volumes using the AWS CLI.
- Understood the purpose of gp3 SSD storage in AWS.
- Applied resource tagging during creation.
- Verified EBS volume configuration using filters.
- Learned the relationship between EBS volumes and Availability Zones.

---

## Best Practices

- Use descriptive tags to simplify resource management.
- Choose the appropriate volume type based on workload requirements.
- Create volumes in the same Availability Zone as the target EC2 instance.
- Monitor storage utilization and performance regularly.
- Remove unused EBS volumes to avoid unnecessary costs.

---

## Challenge Status

✅ Successfully Completed
