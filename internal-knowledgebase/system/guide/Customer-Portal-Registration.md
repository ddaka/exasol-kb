---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Customer Portal Registration"
summary: "Internal Salesforce procedure for enabling customer user access to the Exasol customer portal and managing license/profile assignment safely."
---

# Customer Portal Registration

## Overview

Customers cannot self-register for the portal. An authorized Exasol user must create/enable access in Salesforce.

## Prerequisites

- Sufficient Salesforce permissions to manage contacts and customer users.
- Customer contact exists (or can be created) and is linked to the correct account.

## Procedure

### 1. Verify or create contact

Ensure the target person exists as a Salesforce contact linked to the correct customer account.

### 2. Open the contact

Search and open the correct contact record.

### 3. Enable customer user

From the contact action menu (top-right), select `Enable Customer User`.

### 4. Select user license

Choose one of:

- `Customer Community` (reserved; limited licenses)
- `Customer Community Login` (floating, granted per login window)

Default choice should be `Customer Community Login` unless explicit approval for full `Customer Community` licensing is available.

### 5. Select profile

- Use `Customer Community – MyExasolUserManager` for MyExasol user managers.
- Otherwise use `Customer Community – MyExasolUser`.

Do not select unrelated profiles.

### 6. Set username format

Append `.exasol` suffix to the username (example: `john.doe@company.com.exasol`).

### 7. Set section and save

Set `Section` to `Community`, then save.

The user receives email instructions to set password and activate access.

## Critical Warning

Do **not** use `Disable Customer User` from contact view for standard deactivation workflows.

That action can permanently remove the customer user account and is not recoverable.

## License Changes / User Maintenance

To change license type or adjust customer user settings:

1. Open the correct contact.
2. Choose `View Customer User`.
3. Update license/profile or reset password as required.
