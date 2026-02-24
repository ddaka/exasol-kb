---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS - How to find an Instance from connection string"
summary: "Resolve SaaS EC2 instances from a database connection string using DNS lookup and AWS EC2 filtering."
---
# SaaS - How to find an Instance from connection string

## Purpose

Map a customer DB connection endpoint to one or more backing EC2 instances.

## Prerequisites

- AWS credentials.
- Database connection string / hostname.

## Procedure

1. Resolve the DB hostname to IPs:

```bash
nslookup <db-connection-hostname>
```

Example:

```sql
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   <id>.<clusters-env>.exasol.com
Address: 18.184.63.114
Name:   <id>.<clusters-env>.exasol.com
Address: 3.123.114.186
Name:   <id>.<clusters-env>.exasol.com
Address: 3.64.213.134
Name:   <id>.<clusters-env>.exasol.com
Address: 3.69.208.143
```

2. Authenticate to AWS account (if needed):
- [`saas-login-to-sso.md`](../troubleshoot/saas-login-to-sso.md)

3. In EC2 console, filter instances by the resolved `Public IPv4 address`.

4. Identify the matching instance(s) and continue with connection workflow:
- [`saas-how-to-connect-to-the-instance.md`](saas-how-to-connect-to-the-instance.md)

## Notes

- A single connection hostname may resolve to multiple IPs.
- Validate region/account context before acting on instances.
