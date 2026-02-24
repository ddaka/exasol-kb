---
tool_name: cos
doc_type: guide
category: system
title: "How to create EXA_PROUST_MONITOR user"
summary: "Recreate EXA_PROUST_MONITOR after restore from older versions so Admin UI Health Panel can access required dictionary views."
---
# How to create EXA_PROUST_MONITOR user

## Purpose

Recreate internal monitoring user `EXA_PROUST_MONITOR` when missing after restore from older versions.

## When this is required

- Exasol 2025.1.x and later normally include this user by default.
- After restoring a database from an older release, this user may be absent.
- Missing user can impact Admin UI Health Panel visibility.

## Prerequisites

- Access to COS and database admin privileges.
- Ability to read `/exa/etc/EXAConf`.

## 1) Decode monitor user password from EXAConf

```bash
echo -n "$(grep ExaMonitorUserPassword /exa/etc/EXAConf | awk '{print $3}')" | base64 -d
```

Store the decoded password temporarily and handle it as sensitive data.

## 2) Create DB user and grants

```sql
CREATE USER EXA_PROUST_MONITOR IDENTIFIED BY "<plain_text_password>";
GRANT CREATE SESSION TO EXA_PROUST_MONITOR;
GRANT SELECT ANY DICTIONARY TO EXA_PROUST_MONITOR;
```

## 3) Reset rapid partition process

```bash
cosrst $(cosps -N | grep rapid | awk '{print $1}') <node_id>
```

Use the correct `<node_id>` for your target environment.

## Validation

- Confirm login works for `EXA_PROUST_MONITOR`.
- Confirm dictionary access for required monitoring queries.
- Verify Health Panel data is populated.


