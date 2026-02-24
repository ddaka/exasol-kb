---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - SSL Cert Time Expiry"
summary: "SSL certification is handled by AWS Certificate Manager."
---
# SaaS - SSL Cert Time Expiry

## Overview

SSL certification is handled by AWS Certificate Manager.

What is AWS CM?

AWS Certificate Manager (ACM) handles the complexity of creating, storing, and renewing public and private SSL/TLS X.509 certificates and keys that protect your AWS websites and applications.

## Procedure

Approximately 60 days before the certificate's expiration, ACM begins the process for managed renewal for ACM certificates. ACM tries to validate each domain name included in the certificate, and after all the domain names associated are validated, the ACM certificate is renewed.

If ACM is unable to renew the certificate after 15 days, AWS admins will receive an email in the shared mailbox (**aws-admin-ct**) with further instructions on how to manually fix the renewal problem. This process differs depending on how the certificate was originally validated.

If support team receives an alert about the certificate has not renewed, please escalate it to Production Engineering team.
