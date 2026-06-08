# Day 03 - Create an AWS Subnet in the Default VPC

## Overview

This challenge focuses on creating a subnet within the default Amazon VPC using the AWS CLI. Subnets are a fundamental networking component in AWS that allow you to segment and organize resources within a VPC.

The objective is to create a subnet named `xfusion-subnet` under the default VPC in the `us-east-1` region and verify its successful creation.

---

## Objective

- Identify the default VPC in the `us-east-1` region
- Create a subnet named `xfusion-subnet`
- Associate the subnet with the default VPC
- Assign an appropriate CIDR block
- Deploy the subnet in an Availability Zone
- Verify the subnet configuration

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon VPC | Provides network isolation within AWS |
| Amazon EC2 | Manage VPC networking resources |
| Subnet | Segment resources within a VPC |
| AWS CLI | Interact with AWS services |

---

## Implementation

### Identify the Default VPC

Retrieve the VPC ID of the default VPC in the `us-east-1` region.

```bash
aws ec2 describe-vpcs \
    --filters "Name=isDefault,Values=true" \
    --region us-east-1 \
    --query 'Vpcs[0].VpcId' \
    --output text
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-vpcs` | Lists all VPCs in the account |
| `--filters "Name=isDefault,Values=true"` | Returns only the default VPC |
| `--region us-east-1` | Executes the command in the specified region |
| `--query 'Vpcs[0].VpcId'` | Extracts only the VPC ID |
| `--output text` | Displays the output in plain text format |

Expected Output:

```text
vpc-0d65bc234b18e0f83
```

---

### Create the Subnet

Create a subnet named `xfusion-subnet` within the default VPC.

```bash
aws ec2 create-subnet \
    --vpc-id vpc-0d65bc234b18e0f83 \
    --cidr-block 172.31.96.0/20 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-subnet}]' \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `create-subnet` | Creates a new subnet within a VPC |
| `--vpc-id` | Specifies the VPC where the subnet will be created |
| `--cidr-block 172.31.96.0/20` | Defines the IP address range for the subnet |
| `--availability-zone us-east-1a` | Places the subnet in the specified Availability Zone |
| `--tag-specifications` | Assigns tags during resource creation |
| `Key=Name,Value=xfusion-subnet` | Sets the subnet name tag |
| `--region us-east-1` | Creates the subnet in the required region |

Expected Output:

```json
{
  "Subnet": {
    "SubnetId": "subnet-0dcdbe8408e2e5e3e",
    "VpcId": "vpc-0d65bc234b18e0f83",
    "State": "available",
    "AvailabilityZone": "us-east-1a",
    "CidrBlock": "172.31.96.0/20"
  }
}
```

---

## Verification

### Verify Subnet Creation

Retrieve the subnet details using the assigned Name tag.

```bash
aws ec2 describe-subnets \
    --filters "Name=tag:Name,Values=xfusion-subnet" \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-subnets` | Retrieves subnet information |
| `--filters` | Filters the output based on specific criteria |
| `Name=tag:Name,Values=xfusion-subnet` | Searches for the subnet using its Name tag |
| `--region us-east-1` | Executes the command in the specified region |

Expected Output:

```json
{
  "Subnets": [
    {
      "SubnetId": "subnet-0dcdbe8408e2e5e3e",
      "VpcId": "vpc-0d65bc234b18e0f83",
      "State": "available",
      "AvailabilityZone": "us-east-1a",
      "CidrBlock": "172.31.96.0/20",
      "Tags": [
        {
          "Key": "Name",
          "Value": "xfusion-subnet"
        }
      ]
    }
  ]
}
```

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Locate Default VPC  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create Subnet       │
│ xfusion-subnet      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Assign CIDR Block   │
│ 172.31.96.0/20      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Deploy to AZ        │
│ us-east-1a          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Subnet       │
│ Configuration       │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-vpcs` | Retrieve VPC information |
| `aws ec2 create-subnet` | Create a subnet within a VPC |
| `aws ec2 describe-subnets` | Verify subnet configuration |

---

## Key Learnings

- Learned how to identify the default VPC using AWS CLI.
- Created a subnet within an existing VPC.
- Assigned a CIDR block to define the subnet's IP range.
- Applied tags during resource creation for easier management.
- Verified subnet configuration using AWS CLI commands.

---

## Best Practices

- Use meaningful resource names through tagging.
- Plan CIDR ranges carefully to avoid IP address conflicts.
- Organize resources across Availability Zones for high availability.
- Verify networking resources immediately after creation.
- Maintain consistent naming conventions across AWS resources.

---

## Challenge Status

✅ Successfully Completed
