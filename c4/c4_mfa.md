---
tool_name: c4
doc_type: guide
category: c4 MFA
title: "c4 AWS Multi-Factor Authentication"
summary: "c4 supports AWS Multi-Factor Authentication (MFA) for enhanced security when accessing AWS resources."
---
# c4 AWS Multi-Factor Authentication

## Overview

c4 supports AWS Multi-Factor Authentication (MFA) for enhanced security when accessing AWS resources.

## When MFA is Required

Your organization may require MFA when:
- Accessing production AWS accounts
- Performing privileged operations
- Using federated AWS access
- Compliance regulations mandate it

## MFA Methods

### Method 1: AWS CLI Configuration

Set up MFA through AWS CLI:

```bash
aws configure set aws_access_key_id YOUR_ACCESS_KEY
aws configure set aws_secret_access_key YOUR_SECRET_KEY
aws configure set aws_session_token YOUR_SESSION_TOKEN
```

### Method 2: Environment Variables

Set MFA credentials via environment variables:

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
export AWS_SESSION_TOKEN="YOUR_SESSION_TOKEN"
```

### Method 3: AWS Profile with MFA

Create AWS profile with MFA device:

**~/.aws/config**:
```ini
[profile mfa]
region = us-east-1
mfa_serial = arn:aws:iam::123456789012:mfa/username
```

**~/.aws/credentials**:
```ini
[mfa]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

Use profile with c4:
```bash
AWS_PROFILE=mfa c4 ls
```

## Getting Session Token

### Using AWS CLI

```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code 123456 \
  --duration-seconds 43200
```

**Output**:
```json
{
  "Credentials": {
    "AccessKeyId": "ASIAIOSFODNN7EXAMPLE",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    "SessionToken": "FwoGZXIvYXdzEH...",
    "Expiration": "2025-01-15T12:00:00Z"
  }
}
```

### Set Credentials

```bash
export AWS_ACCESS_KEY_ID="ASIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_SESSION_TOKEN="FwoGZXIvYXdzEH..."
```

## Using MFA with c4

### List Deployments

```bash
AWS_SESSION_TOKEN="YOUR_TOKEN" c4 ls
```

### Create Deployment

```bash
AWS_SESSION_TOKEN="YOUR_TOKEN" c4 apply -P -i ./config
```

### Connect to Deployment

```bash
AWS_SESSION_TOKEN="YOUR_TOKEN" c4 connect -i PLAY_ID -s cos
```

## Session Token Expiration

### Check Expiration

```bash
aws sts get-caller-identity
```

If expired, you'll see:
```
An error occurred (ExpiredToken) when calling the GetCallerIdentity operation
```

### Refresh Token

```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code NEW_TOKEN_CODE \
  --duration-seconds 43200
```

## Session Duration

| Duration | Seconds | Use Case |
|----------|---------|----------|
| **1 hour** | 3600 | Quick tasks |
| **4 hours** | 14400 | Normal operations |
| **12 hours** | 43200 | Long operations |
| **24 hours** | 86400 | Max allowed (role-based) |

## Automation with MFA

### Script with Token Refresh

```bash
#!/bin/bash
# refresh-mfa.sh

read -p "Enter MFA code: " MFA_CODE

CREDENTIALS=$(aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/$(whoami) \
  --token-code "$MFA_CODE" \
  --duration-seconds 43200)

export AWS_ACCESS_KEY_ID=$(echo "$CREDENTIALS" | jq -r '.Credentials.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo "$CREDENTIALS" | jq -r '.Credentials.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo "$CREDENTIALS" | jq -r '.Credentials.SessionToken')

echo "MFA session activated. Valid for 12 hours."
```

Usage:
```bash
source ./refresh-mfa.sh
c4 ls
```

### Using assume-role with MFA

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/ExasolAdmin \
  --role-session-name exasol-session \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code 123456
```

## Troubleshooting

### "InvalidClientTokenId" Error

**Problem**: Invalid access key or secret key

**Solution**:
```bash
# Verify credentials
aws sts get-caller-identity

# Check environment variables
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
```

### "ExpiredToken" Error

**Problem**: Session token expired

**Solution**: Get new session token
```bash
aws sts get-session-token \
  --serial-number YOUR_MFA_ARN \
  --token-code NEW_CODE
```

### "Access Denied" with Valid Token

**Problem**: IAM permissions insufficient

**Solution**: Check IAM policy includes required permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:*",
      "sts:GetSessionToken"
    ],
    "Resource": "*"
  }]
}
```

### MFA Device Not Found

**Problem**: MFA serial number incorrect

**Solution**: Get correct MFA ARN
```bash
aws iam list-mfa-devices
```

## Best Practices

**Use long session durations**
- Request 12-hour tokens for deployment work
- Avoid repeated authentication

**Store credentials securely**
- Never commit tokens to git
- Use environment variables
- Clear after session

**Automate token refresh**
- Create helper scripts
- Integrate with workflow tools
- Set up notifications before expiration

**Use IAM roles when possible**
- Preferred over long-lived credentials
- Automatic rotation
- Better security

**Document MFA requirements**
- Team onboarding
- Runbooks
- Access procedures

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Best Practices](c4_best_practices.md)
- [AWS MFA Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)
- [AWS STS Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)
