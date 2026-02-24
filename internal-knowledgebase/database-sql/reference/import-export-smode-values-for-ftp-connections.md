---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "IMPORT/EXPORT smode Values for FTP Connections"
summary: "Internal reference for FTP smode flags controlling TLS/SSL behavior in Exasol IMPORT/EXPORT connections."
---

# IMPORT/EXPORT smode Values for FTP Connections

## Internal Use Only

This document is intended for internal debugging and temporary workarounds.

## Syntax

Provide `smode` flags before the FTP URL in the connection string:

```sql
EXPORT ... AT 'smode=<OPTION>[,<OPTION>],ftp://...'
```

Example:

```sql
EXPORT ... AT 'smode=USESSL_ALL,ftp://...'
```

You can combine multiple comma-separated options, typically one from each option group below.

## Option Group 1: SSL Usage Mode (`CURLOPT_USE_SSL`)

- `USESSL_NONE`: do not attempt SSL/TLS.
- `USESSL_TRY`: try SSL/TLS, continue without it if possible.
- `USESSL_CONTROL`: require SSL/TLS for control channel.
- `USESSL_ALL`: require SSL/TLS for control and data channels.

Default: `USESSL_CONTROL`

Runtime behavior notes:

- On SSL error, transfer may retry with `USESSL_NONE`.
- On data transfer error, transfer may retry with `USESSL_ALL`.

Reference: <https://curl.se/libcurl/c/CURLOPT_USE_SSL.html>

## Option Group 2: AUTH Method Preference (`CURLOPT_FTPSSLAUTH`)

- `FTPAUTH_DEFAULT`: let libcurl decide.
- `FTPAUTH_SSL`: try `AUTH SSL` first, then `AUTH TLS`.
- `FTPAUTH_TLS`: try `AUTH TLS` first, then `AUTH SSL`.

Default: `FTPAUTH_DEFAULT`

Reference: <https://curl.se/libcurl/c/CURLOPT_FTPSSLAUTH.html>

## Option Group 3: Clear Command Channel (`CURLOPT_FTP_SSL_CCC`)

- `CCC_NONE`: do not use CCC.
- `CCC_PASSIVE`: wait for server-initiated shutdown.
- `CCC_ACTIVE`: initiate shutdown and wait for reply.

Default: `CCC_NONE`

Reference: <https://curl.se/libcurl/c/CURLOPT_FTP_SSL_CCC.html>
