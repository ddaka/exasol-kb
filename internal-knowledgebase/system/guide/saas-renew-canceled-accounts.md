---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS - renew canceled accounts"
summary: "After 30 days trial period or promotional credits are finished, trial account is canceled. Sometimes sales ask to extend trial."
---
# SaaS - renew canceled accounts

## Purpose

Reactivate canceled trial accounts in Chargebee when Sales requests an extension.

## Prerequisites

- Access to Chargebee production account.
- Support ticket from Sales.
- Account UUID, new trial expiry date, and promotional credit amount.

## Procedure

1. Log in to Chargebee production account.

2. Reactivate canceled subscription:
- Open `Subscriptions`.
- Search using account UUID.
- Open the subscription record.
- Use `Reactivate canceled account` and set new expiry date.

3. Update customer billing settings and credits:
- Open `Customers`.
- Search by account UUID and open customer record.
- Edit customer and enable `billing_info_optional` (if required by policy).
- Use `Update Promotional Credits` and apply approved amount.

4. Validate outcome:
- Subscription status is active.
- New expiry date is correct.
- Promotional credit balance matches request.

## Additional References

- <https://www.chargebee.com/docs/2.0/reactivation.html>
