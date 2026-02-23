---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_configure
  - db_info
  - db_start
technical_entities:
  - database
  - parameters
summary: >
  How to manage Exasol database parameters — add, replace, and remove
  key-value parameters via confd_client db_configure, and view current
  parameters with db_info.
---

# Manage Database Parameters

Parameters use key-value format: `"-parameter":"value"`. Undefined parameters
use hard-coded defaults.

> **Warning:** Invalid parameters prevent the database from starting. Contact
> Support before adding or changing parameters.

## Add Parameters

**Prerequisite:** Database must be stopped.

```bash
confd_client db_configure db_name: MY_DATABASE \
  params_add: '[-forceProtocolEncryption=1, -oidcProviderClientSecret=abcd]'
```

## Replace Parameters

**Prerequisite:** Database must be stopped. Parameters must already exist.

```bash
confd_client db_configure db_name: MY_DATABASE \
  params: '[-forceProtocolEncryption=0]'
```

## Remove Parameters

```bash
confd_client db_configure db_name: MY_DATABASE \
  params_delete: '[-forceProtocolEncryption]'
```

## View Current Parameters

```bash
confd_client db_info db_name: MY_DATABASE -j | jq -r '.config.params'
```

After any parameter change, restart the database:

```bash
confd_client db_start db_name: MY_DATABASE
```
