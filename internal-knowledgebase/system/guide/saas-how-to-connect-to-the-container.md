---
tool_name: confd_client
doc_type: guide
category: system
title: "SaaS - How to Connect to the Container"
summary: "Access SaaS container namespace using host shell, direct SSM document, or helper CLI functions."
---
# SaaS - How to Connect to the Container

## Purpose

Show supported ways to reach the SaaS container namespace.

## Prerequisites

- AWS credentials and permissions for target account/region.
- AWS SSM plugin installed locally.
- Target instance ID or DB UUID + node mapping.

## Option 1: connect via instance shell

1. Connect to the EC2 instance first:
- [`saas-how-to-connect-to-the-instance.md`](saas-how-to-connect-to-the-instance.md)

2. Switch into container namespace over localhost SSH (example path):

```bash
cd ~/.ccc/play/local/<id>/main/<nr>/root/
sudo su
cd root/.ssh
ssh -i id_rsa localhost -p20002
```

## Option 2: direct SSM container session

1. Get instance ID from AWS console or CLI.

2. Start a direct container session:

```bash
aws ssm start-session --target <instance_id> --region <region> --document-name Container
```

This method avoids browser session timeouts from AWS console UI.

## Option 3: connect by DB UUID and node alias

Use helper functions to resolve instance IDs from stack resources/tags.

1. Add helper functions to your shell profile (one-time setup):

```bash
function aws-instance() {
  node="${4:-n11}"
  aws cloudformation --profile "$1" --region "$2" \
    describe-stack-resources --stack-name "$3" --logical-resource-id "$node" \
    | jq -r '.StackResources[0].PhysicalResourceId'
}

function aws-instance-tag() {
  aws ec2 describe-instances --profile "$1" --region "$2" \
    --filters "Name=tag:saas:DBUUID,Values=$3" "Name=tag:aws:cloudformation:logical-id,Values=$4" \
    --query 'Reservations[*].Instances[*].InstanceId' --output text
}

function aws-ssh() {
  instance="$(aws-instance-tag "$1" "$2" "$3" "$4")"
  aws ssm start-session --profile "$1" --target "$instance" --region "$2" --document-name Container
}
```

2. Connect by profile, region, DB UUID, and node id:

```bash
aws-ssh <sso-profile> <region> <db_uuid> <node_number>
```

## External reference

- AWS SSM plugin installation:
  <https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html>
