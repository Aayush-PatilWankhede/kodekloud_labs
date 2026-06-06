# Day 02 - Create and Configure an AWS Security Group

## Overview

This challenge focuses on creating and configuring an AWS Security Group within the default VPC. Security Groups act as virtual firewalls that control inbound and outbound traffic for AWS resources such as EC2 instances.

The objective is to create a security group named `datacenter-sg` and configure inbound rules to allow HTTP and SSH access.

---

## Objective

- Identify the default VPC in the `us-east-1` region
- Create a Security Group named `datacenter-sg`
- Add an inbound HTTP rule on port `80`
- Add an inbound SSH rule on port `22`
- Allow access from any IP address (`0.0.0.0/0`)
- Verify the Security Group configuration

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Create and manage Security Groups |
| Amazon VPC | Provide network isolation and default VPC |
| AWS CLI | Interact with AWS services |
| Security Groups | Control inbound and outbound traffic |

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
| `describe-vpcs` | Lists available VPCs |
| `--filters "Name=isDefault,Values=true"` | Returns only the default VPC |
| `--region us-east-1` | Executes the command in the specified region |
| `--query 'Vpcs[0].VpcId'` | Extracts only the VPC ID from the response |
| `--output text` | Displays the output as plain text |

Expected Output:

```text
vpc-0c63a10e647db1dc7
```

---

### Create the Security Group

Create a Security Group named `datacenter-sg` inside the default VPC.

```bash
aws ec2 create-security-group \
    --group-name datacenter-sg \
    --description "Security group for Nautilus App Servers" \
    --vpc-id vpc-0c63a10e647db1dc7 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `create-security-group` | Creates a new Security Group |
| `--group-name datacenter-sg` | Specifies the Security Group name |
| `--description` | Provides a description for the Security Group |
| `--vpc-id` | Associates the Security Group with the specified VPC |
| `--region us-east-1` | Creates the Security Group in the required region |

Expected Output:

```json
{
  "GroupId": "sg-0b6a57e04b1303f44",
  "SecurityGroupArn": "arn:aws:ec2:us-east-1:915322164324:security-group/sg-0b6a57e04b1303f44"
}
```

---

### Add HTTP Inbound Rule

Allow inbound HTTP traffic on port `80` from any IP address.

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-0b6a57e04b1303f44 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `authorize-security-group-ingress` | Adds an inbound rule to a Security Group |
| `--group-id` | Specifies the target Security Group |
| `--protocol tcp` | Allows TCP traffic |
| `--port 80` | Opens HTTP port 80 |
| `--cidr 0.0.0.0/0` | Allows access from any IPv4 address |
| `--region us-east-1` | Executes the command in the specified region |

---

### Add SSH Inbound Rule

Allow inbound SSH traffic on port `22` from any IP address.

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-0b6a57e04b1303f44 \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `authorize-security-group-ingress` | Adds an inbound rule to a Security Group |
| `--group-id` | Specifies the target Security Group |
| `--protocol tcp` | Allows TCP traffic |
| `--port 22` | Opens SSH port 22 |
| `--cidr 0.0.0.0/0` | Allows access from any IPv4 address |
| `--region us-east-1` | Executes the command in the specified region |

---

## Verification

### Verify Security Group Configuration

Retrieve the Security Group details and verify the configured inbound rules.

```bash
aws ec2 describe-security-groups \
    --group-ids sg-0b6a57e04b1303f44 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-security-groups` | Retrieves Security Group information |
| `--group-ids` | Filters the output for the specified Security Group |
| `--region us-east-1` | Executes the command in the required region |

Expected Output:

```json
{
  "GroupName": "datacenter-sg",
  "Description": "Security group for Nautilus App Servers",
  "IpPermissions": [
    {
      "IpProtocol": "tcp",
      "FromPort": 80,
      "ToPort": 80,
      "IpRanges": [
        {
          "CidrIp": "0.0.0.0/0"
        }
      ]
    },
    {
      "IpProtocol": "tcp",
      "FromPort": 22,
      "ToPort": 22,
      "IpRanges": [
        {
          "CidrIp": "0.0.0.0/0"
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
│ Create Security     │
│ Group               │
│ datacenter-sg       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Add HTTP Rule       │
│ Port 80             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Add SSH Rule        │
│ Port 22             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Security     │
│ Group Configuration │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 describe-vpcs` | Retrieve VPC information |
| `aws ec2 create-security-group` | Create a Security Group |
| `aws ec2 authorize-security-group-ingress` | Add inbound Security Group rules |
| `aws ec2 describe-security-groups` | Verify Security Group configuration |

---

## Key Learnings

- Learned how to identify the default VPC using AWS CLI.
- Created and configured Security Groups in AWS.
- Configured HTTP and SSH access using inbound rules.
- Understood how Security Groups act as virtual firewalls.
- Verified Security Group configurations using AWS CLI.

---

## Best Practices

- Follow the principle of least privilege when defining Security Group rules.
- Restrict SSH access to trusted IP ranges whenever possible.
- Use descriptive names and descriptions for Security Groups.
- Regularly audit Security Group rules for unnecessary exposure.
- Verify network configurations after making changes.

---

## Challenge Status

✅ Successfully Completed
