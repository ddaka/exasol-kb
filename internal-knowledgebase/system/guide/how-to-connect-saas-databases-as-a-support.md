---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "How to connect SaaS databases as support"
summary: "Connect to a SaaS customer database as support using AWS SSM port forwarding and monitoring user credentials from Parameter Store."
---
# How to connect SaaS databases as support

## Purpose

Access a SaaS customer database for support tasks without direct inbound network exposure, using AWS SSM tunnel forwarding.

## Prerequisites

- AWS access to relevant SaaS accounts.
- AWS CLI configured.
- SSM Session Manager permissions.
- Database client (for example DBeaver, `EXAplus`, JDBC client).

## 1) Authenticate to AWS

Log in using the standard SSO flow for the target environment/profile.

## 2) Identify target cluster and DB node

- Determine `cluster_uuid` and target database.
- In AWS console (`prod-saas-customers` account), identify an EC2 instance for the target DB node.
- Note instance ID (`i-...`) and stack identifier/tag.

## 3) Open SSM port-forwarding tunnel

```shell
aws ssm start-session \
  --target <instance_id> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["8563"],"localPortNumber":["8563"]}' \
  --profile <aws_profile>
```

Keep this session open while connecting.

## 4) Retrieve DB support user password

In SaaS management account (`prod-saas`, correct region), open **Systems Manager > Parameter Store**.

- Search by stack identifier.
- Retrieve the value for support/monitoring DB user (for example `exa_monitoring_user`).

## 5) Connect with your DB client

Use:

- Host: `127.0.0.1`
- Port: `8563`
- User: support/monitoring user
- Password: value from Parameter Store

## Notes

- Use least-privilege account/profile for access.
- Close SSM session when support activity is complete.

## Reference

- <https://aws.amazon.com/blogs/aws/new-port-forwarding-using-aws-system-manager-sessions-manager/>


