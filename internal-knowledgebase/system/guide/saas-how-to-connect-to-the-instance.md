---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS - How to Connect to the Instance"
summary: "Connect to SaaS EC2 instances either through AWS Console Session Manager or AWS CLI."
---
# SaaS - How to Connect to the Instance

## Purpose

Show standard ways to connect to SaaS instances.

## Prerequisites

- AWS credentials and permissions for target account/region.
- Target instance ID (or ability to locate it in EC2).
- For CLI method: AWS SSM plugin installed.

Connection methods:

- AWS Console
- AWS CLI

## Method 1: AWS Console

1. Open AWS Console and go to `EC2`.
2. Select the target instance.
3. Click `Connect`.
4. Use Session Manager based connection flow.

## Method 2: AWS CLI (recommended for repeatable access)

1. Get the instance ID from EC2 console or CLI.
2. Start a session:

```bash
aws ssm start-session --target <instance-id> --region <region>
```

## External reference

- AWS SSM plugin installation:
  <https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html>
