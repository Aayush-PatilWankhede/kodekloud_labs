# Day 06 - Launch an Amazon EC2 Instance Using AWS CLI

## Overview

This challenge focuses on provisioning an Amazon EC2 instance using the AWS CLI. Amazon EC2 provides scalable virtual servers in the AWS Cloud and is one of the core services used to deploy applications and infrastructure.

The objective is to create a new EC2 instance named `nautilus-ec2`, generate an RSA key pair for secure access, attach the default security group, and launch the instance using the latest Amazon Linux AMI.

---

## Objective

- Create a new RSA key pair named `nautilus-kp`
- Secure the generated private key file
- Retrieve the latest Amazon Linux AMI
- Launch an EC2 instance named `nautilus-ec2`
- Use the `t2.micro` instance type
- Attach the default security group
- Verify successful instance creation

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Launch and manage virtual servers |
| Amazon Machine Images (AMI) | Provide operating system templates |
| AWS CLI | Interact with AWS services |
| EC2 Key Pairs | Secure SSH authentication |
| Security Groups | Control network access to instances |

---

## Implementation

### Create an RSA Key Pair

Generate a new RSA key pair and save the private key locally.

```bash
aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --region us-east-1 \
    --query 'KeyMaterial' \
    --output text > nautilus-kp.pem
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `create-key-pair` | Creates a new EC2 key pair |
| `--key-name nautilus-kp` | Specifies the key pair name |
| `--key-type rsa` | Creates an RSA-based key pair |
| `--region us-east-1` | Creates the key pair in the specified region |
| `--query 'KeyMaterial'` | Extracts only the private key content |
| `--output text` | Outputs the key material as plain text |
| `> nautilus-kp.pem` | Saves the private key locally |

---

### Secure the Private Key

Restrict access to the PEM file.

```bash
chmod 400 nautilus-kp.pem
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `chmod 400` | Grants read permission only to the file owner |
| `nautilus-kp.pem` | Private key file |

This ensures the key can be used securely for SSH access.

---

### Retrieve the Latest Amazon Linux AMI

Fetch the latest available Amazon Linux AMI ID.

```bash
aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=al2023-ami-*" \
    "Name=architecture,Values=x86_64" \
    "Name=root-device-type,Values=ebs" \
    --query 'Images | sort_by(@,&CreationDate)[-1].ImageId' \
    --output text \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-images` | Retrieves available AMIs |
| `--owners amazon` | Limits results to official Amazon AMIs |
| `Name=name,Values=al2023-ami-*` | Filters Amazon Linux 2023 images |
| `Name=architecture,Values=x86_64` | Selects x86_64 architecture |
| `Name=root-device-type,Values=ebs` | Selects EBS-backed AMIs |
| `sort_by(@,&CreationDate)[-1]` | Returns the newest AMI |
| `--output text` | Displays only the AMI ID |

Example Output:

```text
ami-xxxxxxxxxxxxxxxxx
```

---

### Launch the EC2 Instance

Create the EC2 instance using the retrieved Amazon Linux AMI.

```bash
aws ec2 run-instances \
    --image-id ami-xxxxxxxxxxxxxxxxx \
    --instance-type t2.micro \
    --key-name nautilus-kp \
    --security-groups default \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]' \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `run-instances` | Launches a new EC2 instance |
| `--image-id` | Specifies the Amazon Linux AMI |
| `--instance-type t2.micro` | Launches a t2.micro instance |
| `--key-name nautilus-kp` | Associates the RSA key pair |
| `--security-groups default` | Attaches the default security group |
| `--tag-specifications` | Assigns tags during creation |
| `Key=Name,Value=nautilus-ec2` | Sets the EC2 instance name |
| `--region us-east-1` | Launches the instance in the required region |

Expected Output:

```json
{
  "Instances": [
    {
      "InstanceId": "i-xxxxxxxxxxxxxxxxx",
      "InstanceType": "t2.micro",
      "State": {
        "Name": "pending"
      }
    }
  ]
}
```

---

## Verification

### Verify Instance Creation

Retrieve details of the newly created instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=nautilus-ec2" \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `describe-instances` | Retrieves EC2 instance information |
| `--filters` | Filters results based on criteria |
| `Name=tag:Name,Values=nautilus-ec2` | Searches by instance Name tag |
| `--region us-east-1` | Executes the command in the specified region |

Expected Output:

```json
{
  "Reservations": [
    {
      "Instances": [
        {
          "InstanceType": "t2.micro",
          "KeyName": "nautilus-kp",
          "State": {
            "Name": "running"
          }
        }
      ]
    }
  ]
}
```

The important fields are:

```json
{
  "InstanceType": "t2.micro",
  "KeyName": "nautilus-kp",
  "State": {
    "Name": "running"
  }
}
```

which confirm that the EC2 instance has been launched successfully.

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create RSA Key Pair │
│ nautilus-kp         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Secure PEM File     │
│ chmod 400           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Retrieve Latest     │
│ Amazon Linux AMI    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Launch EC2 Instance │
│ nautilus-ec2        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Attach Default      │
│ Security Group      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Instance     │
│ Status              │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws ec2 create-key-pair` | Create a new EC2 key pair |
| `chmod 400 nautilus-kp.pem` | Secure the private key file |
| `aws ec2 describe-images` | Retrieve Amazon Linux AMIs |
| `aws ec2 run-instances` | Launch an EC2 instance |
| `aws ec2 describe-instances` | Verify EC2 instance details |

---

## Key Learnings

- Learned how to create EC2 key pairs using AWS CLI.
- Understood how to securely manage PEM files.
- Retrieved the latest Amazon Linux AMI dynamically.
- Launched an EC2 instance with a specific instance type.
- Verified EC2 resources using AWS CLI commands.

---

## Best Practices

- Never commit PEM files to source control repositories.
- Apply least-privilege permissions to private key files.
- Use official Amazon-provided AMIs whenever possible.
- Tag cloud resources consistently for easier management.
- Verify resources immediately after provisioning.

---

## Challenge Status

✅ Successfully Completed
