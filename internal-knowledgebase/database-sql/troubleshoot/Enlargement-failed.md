---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Enlargement failed"
summary: "Troubleshooting failed database enlargement operations, including the temporary enlargeCluster workaround."
---
# Enlargement failed

## Problem

A database enlargement operation fails, or the database starts and then stops shortly after enlargement.

## Symptoms

- Enlargement workflow does not complete successfully.
- `pdd` logs report node-count mismatch or related consistency errors.
- Database briefly starts and then shuts down again.

## Diagnostic Procedure

1. Inspect database server logs, focusing on `pdd` messages.
1. Confirm expected node count and enlargement target configuration.
1. Verify whether recent cluster topology changes were applied completely.

## Workaround for node-count mismatch

If `pdd` reports a wrong number of nodes and the start/stop pattern above is observed, apply this temporary sequence:

1. Set the `enlargeCluster` flag for the affected database.
1. Start the database.
1. Stop the database.
1. Remove the `enlargeCluster` flag.
1. Start the database again.

Use standard internal tooling/procedure for setting and removing database flags.

## Escalation

Escalate with collected logs when:

- The workaround does not stabilize startup.
- Node-count mismatch persists after configuration validation.
- Repeated enlargement attempts fail.

## Result

The incident is resolved when the database starts normally after enlargement and remains stable.

---

_We welcome feedback on this troubleshooting article._
