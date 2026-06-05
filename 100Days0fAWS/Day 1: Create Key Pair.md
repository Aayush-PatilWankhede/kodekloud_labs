# Day 01 - Create an AWS EC2 Key Pair Using AWS CLI

## Overview

This challenge focuses on creating an AWS EC2 Key Pair using the AWS CLI. EC2 Key Pairs are used to securely connect to Linux instances and form an essential part of AWS infrastructure management.

---

## Objective

- Create an EC2 Key Pair named `devops-kp`
- Use the RSA encryption algorithm
- Create the resource in the `us-east-1` region
- Secure the generated private key
- Verify AWS account access

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon EC2 | Create and manage Key Pairs |
| AWS STS | Verify AWS identity |
| AWS CLI | Interact with AWS services |
| Linux | Manage file permissions |

---

## Implementation

### Verify AWS Identity

Before creating resources, verify the currently authenticated AWS account.

```bash
aws sts get-caller-identity
```

### Create RSA Key Pair

Create a new EC2 Key Pair named `devops-kp` and save the private key locally.

```bash
aws ec2 create-key-pair \
    --key-name devops-kp \
    --key-type rsa \
    --region us-east-1 \
    --query 'KeyMaterial' \
    --output text > devops-kp.pem
```

### Secure the Private Key

Restrict permissions so only the owner can access the key.

```bash
chmod 400 devops-kp.pem
```

---

## Verification

### Verify AWS Account

```bash
aws sts get-caller-identity
```

Expected Output:

```json
{
  "UserId": "XXXXXXXXXXXX",
  "Account": "XXXXXXXXXXXX",
  "Arn": "arn:aws:iam::XXXXXXXXXXXX:user/username"
}
```

### Verify Key Pair Creation

```bash
aws ec2 describe-key-pairs \
    --key-names devops-kp \
    --region us-east-1
```

Expected Output:

```json
{
  "KeyPairs": [
    {
      "KeyName": "devops-kp",
      "KeyType": "rsa"
    }
  ]
}
```

---

## Workflow

```text
┌─────────────────────┐
│    AWS CLI User     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Identity     │
│ AWS STS             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create EC2 Key Pair │
│ devops-kp           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Generate PEM File   │
│ devops-kp.pem       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Secure Permissions  │
│ chmod 400           │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws sts get-caller-identity` | Verify active AWS identity |
| `aws ec2 create-key-pair` | Create an EC2 Key Pair |
| `chmod 400 devops-kp.pem` | Secure the PEM file |
| `aws ec2 describe-key-pairs` | Verify key pair creation |

---

## Key Learnings

- Learned how to create EC2 Key Pairs using AWS CLI.
- Understood the purpose of RSA-based authentication in AWS.
- Practiced securing PEM files using Linux permissions.
- Verified AWS account access using STS.

---

## Best Practices

- Never commit `.pem` files to Git repositories.
- Apply least-privilege file permissions to private keys.
- Verify AWS credentials before provisioning resources.
- Use descriptive naming conventions for cloud resources.
- Store private keys securely using a secrets management solution.

---

## Challenge Status

✅ Successfully Completed
