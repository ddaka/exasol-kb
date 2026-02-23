---
tool_name: confd_client
doc_type: reference
category: troubleshooting
technical_entities:
  - error codes
  - error catalog
  - troubleshooting
summary: >
  How to interpret and look up Exasol error codes using the Exasol Error Catalog.
---

# Error Codes

Various Exasol components generate error codes when operations fail. Use these
codes to identify and troubleshoot the cause.

## Exasol Error Catalog

All error codes are documented in the official catalog:

**URL:** [https://error-catalog.exasol.com/](https://error-catalog.exasol.com/)

> The catalog is still in development and does not have an internal search
> function.

## Looking Up Error Codes

### Method 1: Search Engine

Search for the error code using Google or Bing, e.g.:

```
site:error-catalog.exasol.com E-MPVG-1
```

### Method 2: Direct URL

Navigate directly by appending the error code to the URL:

```
https://error-catalog.exasol.com/error-codes/e-mpvg-1.html
```

## Error Code Format

Exasol error codes follow the pattern:

```
E-<COMPONENT>-<NUMBER>
```

Where `<COMPONENT>` identifies the Exasol module that generated the error and
`<NUMBER>` is the specific error identifier.
