---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "INTERNAL - Backup Measure from PDDServer"
summary: "This article should help to understand PDD statistics printed for each backup run."
---
# INTERNAL - Backup Measure from PDDServer

## Overview

This article should help to understand PDD statistics printed for each backup run.

## Explanation

 Those statistics will be printed after the backup has finished. The backup process consists of three stages: READ, WRITE, VALIDATE. In order to find the statistics run the following command on the logical database node 0:

```python
# ssh n11
# cd /d02_data/DB_NAME/log/process/
# grep -i backup *Pdd*
```

Search  for the section '**backupMeasure**':

```python
01.12 21:16:50.986 EXAOP_NOTICE Backup finished: DB1/id_266/level_0/node_0/backup_202012010000
01.12 21:16:50.988  BACKUP: backupMeasure
01.12 21:16:50.988    BACKUP: READ REAL TIME: 18226.637 s count: 191004 avg: 0.095425 s
01.12 21:16:50.988    BACKUP: READ VALUE: 2995598.742 M count: 191004 avg: 15.683434 M
01.12 21:16:50.988    BACKUP: WRITE REAL TIME: 37438.311 s count: 2259900 avg: 0.016566 s
01.12 21:16:50.988    BACKUP: WRITE VALUE: 2995598.742 M count: 2259900 avg: 1.325544 M
01.12 21:16:50.988    BACKUP: VALIDATION REAL TIME: 19707.400 s count: 2451486 avg: 0.008038 s
01.12 21:16:50.988    BACKUP: LOCALITYCHECK REAL TIME: 0.000001 s count: 1 avg: 0.000001 s
01.12 21:16:50.988    BACKUP: LIMITATION REAL TIME: 0.229896 s count: 382591 avg: 0.000000 s
01.12 21:16:50.988    BACKUP: FLUSH REAL TIME: 5.732840 s count: 1 avg: 5.732840 s
01.12 21:16:50.988    BACKUP: PREPARE REAL TIME: 15.069487 s count: 1 avg: 15.069487 s
01.12 21:16:50.988    BACKUP: COMPLETE REAL TIME: 76609.167 s count: 1 avg: 76609.167 s
01.12 21:33:59.955    BACKUP: VALIDATION REAL TIME: 1491.576400 s count: 224281 avg: 0.006650 s
01.12 21:33:59.955    BACKUP: LIMITATION REAL TIME: 0.012639 s count: 20116 avg: 0.000000 s
01.12 21:33:59.955    BACKUP: COMPLETE REAL TIME: 76609.167 s count: 1 avg: 76609.167 s
```

Now let's have a look at the interesting timers:

* **READ REAL TIME** in Seconds
* **WRITE REAL TIME** in Seconds
* **VALIDATE REAL TIME** in Seconds

and the interesting data numbers in Megabytes:

* **READ VALUE** in Megabytes
* **WRITE VALUE** in Megabytes
