---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Connect Ruby to Exasol via ODBC"
summary: "Set up Ruby ODBC connectivity to Exasol, including required gem configuration, ODBC DSN settings, and test script references."
---

# Connect Ruby to Exasol via ODBC

## Overview

This guide documents one way to connect Ruby applications to Exasol using ODBC.

Alternative Ruby connectivity options include:

- JRuby with JDBC
- Exasol WebSocket connector

## Prerequisites

- Ruby runtime environment.
- ODBC manager and Exasol ODBC driver installed on the host.
- Network connectivity to Exasol.
- Exasol user credentials and target schema details.

## Setup Artifacts

The following attachment files are referenced by this workflow:

1. [`README.rtf`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/README.rtf): step-by-step setup session notes.
2. [`Gemfile`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/Gemfile): Ruby gem dependencies for `ruby-odbc`.
3. [`.odbc.ini`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/.odbc.ini): DSN example with Exasol-specific options.
4. [`30select.rb`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/30select.rb): sample query test script.
5. [`log_session_01.rtf`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/log_session_01.rtf): main shell session output.
6. [`log_session_02.rtf`](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/log_session_02.rtf): secondary shell session output.

## Configuration Notes

When adapting `.odbc.ini`, verify required Exasol options are set correctly for your environment, including:

- `EXASCHEMA`
- `INTTYPESINRESULTSIFPOSSIBLE`

## Validation

- Confirm DSN connectivity with your ODBC tooling.
- Run the sample Ruby query script and verify result ordering/output.
- Check logs for driver negotiation or authentication issues.

## Notes

This content originated from PoC usage and should be validated against your current Ruby/driver versions before production use.
