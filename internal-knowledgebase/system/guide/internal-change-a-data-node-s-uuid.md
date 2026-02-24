---
tool_name: cos
doc_type: guide
category: system
title: "Internal - Change a data node UUID"
summary: "Internal legacy procedure to change node UUID metadata when a replaced node reuses conflicting UUID values."
---
# Internal - Change a data node UUID

## Purpose

Resolve UUID conflicts when a replaced data node reuses metadata that matches a previously removed node UUID.

## Critical warning

Proceed only if both are true:

- Target node is a reserve node.
- Target node holds no active data.

## Procedure

1. Check node UUID on target node:

```shell
cat /etc/cos/node_uuid
```

2. Stop EXAoperation services as required by your environment.
3. Open `exaopctl` shell:

```shell
exaopctl shell
```

4. Read current UUID in cluster metadata (example node `n0011`):

```python
print root['cluster1']['n0011'].node_unique_id
```

5. Set new UUID value:

```python
root['cluster1']['n0011'].node_unique_id = u'<NEW_UUID>'
```

6. Verify new UUID:

```python
print root['cluster1']['n0011'].node_unique_id
```

7. Commit and exit:

```python
import transaction
transaction.commit()
exit
```

8. Set install flag in node configuration and reinstall/reboot node.

## Validation

- Node rejoins cluster with new UUID.
- No duplicate UUID appears in cluster metadata.
- Node state is healthy after reinstall.

## De-duplication note

For standard node lifecycle operations, use canonical ConfD node docs:

- `documents/cos/confd-node-management.md`


