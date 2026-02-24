---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS - How to get account owner"
summary: "During the incident or maintenance we need to contact account owner to update him. Currently, SaaS is not integrated to Jira and we don't have automatic information about accounts."
---
# SaaS - How to get account owner

## Overview

During the incident or maintenance we need to contact account owner to update him. Currently, SaaS is not integrated to Jira and we don't have automatic information about accounts.

If a SaaS customer will become an enterprise customer, usual onboarding process will be done and contact information will be available in Jira.

## Prerequisites

Access to SaaS production environment.

Customer Account UUID (monitoring alerts will have this account uuid)

## How to get user who created the account

### Step 1

Connect to SaaS prod management container. See how to connect container [playbook](saas-how-to-connect-to-the-container.md)

### Step 2

Use below confd job to get owners:

```bash
confd_client saas_user_list account_uuid: <provide_customer_account_uuid> next: 0 limit: 100 | grep -B5 Owner
```

Instead of `$provide_customer_account_uuid`, one needs to use actual customer account uuid. Please note that account uuid is a case sensitive and tags are in lower case (Datadog tags don't support upper case letters).
