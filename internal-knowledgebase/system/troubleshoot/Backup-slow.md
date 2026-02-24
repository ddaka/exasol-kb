---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Backup stuck/slow"
summary: "A backup is stuck or is very slow."
---
# Backup stuck/slow

## Overview

A backup is stuck or is very slow.

## Explanation

Check pddserver log of logical node 0
- dwad_client sys-nodes DB
- e.g. no progress since 10h

Abort/Cancel backup
- dwad_client abort-backup XXX
- or via EXAoperation

Or restart database (stop/start)  - only if the customer agrees
- dwad_client stop-wait XXX
- dwad_client start-wait XXX
- or via EXAoperation

Check logs (pddserver, logd_collect)
- grep -i backup /d02_data/exa_db1/log/process/*Pdd* (logical node 0)
- logd_collect EXASolution_exa_db1|grep -iE “backup|pdd”
- local backup, remote (FTP, S3, SMB)
- Volume full? “No space left on device/volume”
- Restart the backup if possible
- E.g. creating a level-1 instead of a level-0 and extending the expire time the old level-0 is a valid incident management!
- psh sdfs list vol-ID (check access for all nodes (psh data nodes only) + mgmt. Node)

- Are all segments local? DB nodes are the same as the archive volume nodes?
- Is there enough free disk space, are there old backups that are deleted

## Additional References

Replace this text with helpful links to other information. Otherwise, delete this section and the Additional References section title.


