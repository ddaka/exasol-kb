---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Tableau Online Connection Issues with Exasol SaaS/V8"
summary: "Troubleshooting guidance for Tableau Online connectivity failures caused by outdated Exasol driver compatibility in legacy Tableau connector paths."
---

# Tableau Online Connection Issues with Exasol SaaS/V8

## Symptoms

Tableau Online fails to connect to Exasol SaaS or Exasol v8 environments when using an embedded legacy connector path.

Typical indicators:

- Connection initialization fails before authentication completes.
- TLS/driver compatibility errors.
- OAuth/OpenID-related incompatibility in older driver stacks.

## Cause

A common cause is use of an outdated Exasol ODBC driver version in the Tableau connector chain, which is incompatible with newer Exasol SaaS/v8 security and protocol requirements.

## Mitigation Options

1. Use the newer Exasol JDBC-based Tableau connector where available.
2. Use Tableau Bridge with a compatible connector/driver stack.
3. Avoid legacy connector paths that force outdated ODBC components.
4. Validate TLS and authentication compatibility end-to-end.

## Validation Checklist

- Confirm Tableau connector version in use.
- Confirm Exasol driver version used by that connector path.
- Verify TLS handshake and cipher compatibility.
- Verify authentication method support (including OpenID/OAuth if required).

## Notes

- This issue is typically environmental/connector-version specific.
- Treat older timeline statements about connector availability as historical context only.
