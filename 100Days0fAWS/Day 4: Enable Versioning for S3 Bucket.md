# Day 04 - Enable Versioning for an AWS S3 Bucket

## Overview

This challenge focuses on enabling versioning for an Amazon S3 bucket. S3 Versioning is a critical data protection feature that preserves multiple versions of an object, helping recover data from accidental deletion, overwrites, or application failures.

The objective is to enable versioning on the existing S3 bucket `devops-s3-340` and verify that the configuration has been applied successfully.

---

## Objective

- Enable versioning on the S3 bucket `devops-s3-340`
- Configure the bucket in the `us-east-1` region
- Verify that versioning has been successfully enabled
- Improve data protection and recovery capabilities

---

## Services Used

| Service | Purpose |
|----------|----------|
| Amazon S3 | Object storage service |
| S3 Versioning | Preserve multiple versions of objects |
| AWS CLI | Interact with AWS services |
| AWS S3 API | Configure bucket settings |

---

## Implementation

### Enable Versioning on the S3 Bucket

Enable versioning for the target S3 bucket.

```bash
aws s3api put-bucket-versioning \
    --bucket devops-s3-340 \
    --versioning-configuration Status=Enabled \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `aws s3api` | Access low-level Amazon S3 API operations |
| `put-bucket-versioning` | Configure bucket versioning settings |
| `--bucket devops-s3-340` | Specifies the target S3 bucket |
| `--versioning-configuration Status=Enabled` | Enables bucket versioning |
| `--region us-east-1` | Executes the operation in the specified AWS region |

Once enabled, all future object uploads and modifications will maintain version history.

---

## Verification

### Verify Versioning Status

Check the current versioning status of the bucket.

```bash
aws s3api get-bucket-versioning \
    --bucket devops-s3-340 \
    --region us-east-1
```

#### Command Explanation

| Option | Description |
|----------|-------------|
| `aws s3api` | Access Amazon S3 API operations |
| `get-bucket-versioning` | Retrieve bucket versioning configuration |
| `--bucket devops-s3-340` | Specifies the target bucket |
| `--region us-east-1` | Executes the command in the specified region |

Expected Output:

```json
{
  "Status": "Enabled"
}
```

The important part is:

```json
"Status": "Enabled"
```

which confirms that S3 Versioning has been successfully enabled.

---

## Workflow

```text
┌─────────────────────┐
│     AWS CLI User    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Select S3 Bucket    │
│ devops-s3-340       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Enable Versioning   │
│ Status=Enabled      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Store Future Object │
│ Versions            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Verify Bucket       │
│ Configuration       │
└─────────────────────┘
```

---

## Command Reference

| Command | Description |
|----------|-------------|
| `aws s3api put-bucket-versioning` | Enable or configure S3 bucket versioning |
| `aws s3api get-bucket-versioning` | Verify bucket versioning status |

---

## Key Learnings

- Learned how to enable versioning on an S3 bucket.
- Understood the importance of versioning for data protection.
- Learned how AWS stores multiple versions of objects.
- Verified bucket configuration using AWS CLI.
- Improved disaster recovery capabilities for stored data.

---

## Best Practices

- Enable versioning on critical production buckets.
- Combine versioning with lifecycle policies to control storage costs.
- Use versioning to recover from accidental deletions and overwrites.
- Regularly audit bucket configurations and security settings.
- Enable MFA Delete for highly sensitive data where applicable.

---

## Challenge Status

✅ Successfully Completed
