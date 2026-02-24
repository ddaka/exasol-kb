---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Playbook: Backup - Sizing an archive volume"
summary: "This article will show, how the calculation of an archive volume is done. Because the correct compression rates are only known at runtime, we will calculate the memory to be made..."
---
# Playbook: Backup - Sizing an archive volume

## Overview

This article will show, how the calculation of an archive volume is done. Because the correct compression rates are only known at runtime, we will calculate the memory to be made available from a few assumptions.

## Synopsis

## Backup levels

Generally, the backup concept includes several levels of backups. Following the corresponding backup levels: Level-0 Full-Backup Level-1 Differential-Backup Level-2 Incremental-Backup Level-3 Incremental-Backup Level-4 Incremental-Backup Level-5 Incremental-Backup Level-6 Incremental-Backup Level-7 Incremental-Backup Level-8 Incremental-Backup Level-9 Incremental-Backup According to the definition of these levels, the full backup will save all data, which has been committed to the database until it has been started. The differential backup will save all data, which has been committed since the last full backup. Last but not least, the Incremental backup, which will save all data which has been committed since the last differential. Full > Differential > Incremental.

## Backup expiration

As already shown in the screenshot from above, the expiration time for a backup is already configured. This time means, that the backup will be removed after this time as reached.

**Note:** Please be aware, that this functionality is enabled by default only for local archive volumes. For remote archive volumes, it's required to enable this functionality by using the "cleanvolume" options. The backup process will always delete the expired backup only, if the backup, which just has been created, was finished successfully. If the volume does not inherit enough space to store the new backup, old, and expired backups, will be deleted before it's finished. This only happens, if the archive volume is not sized properly and/or the amount of data in the database has been increased. We're using the following expiration times usually: Level-0 10d Level-1 3d By using this times, it will lead to the situation, that the maximum count of backups is 2xLevel-0 and 3xLevel-1, which is stored within the volume.

## Sizes

Based on the data volume of 5TB we will use some experienced values in order to forecast the needed space for writing backups. Of course, this will change, when importing more data to the database. You will find the values about the RAW and MEM sizes within the database at the table "EXA_GLOBAL_DB_SIZE_DAILY". RAW data means uncompressed (csv) and MEM means compressed data. Furthermore, we're assuming a compression ratio of 2.5 and a usage of 10% for the level-1 backup compared to the level-0 backup. RAW Size: 5TB Compression ratio RAW>MEM: 2.5 Compression ratio Level-0>Level-1:10%

## Calculation

RAW size / compression ratio RAW>MEM = MEM size 5TB/2.5 = 2TB 2TB == 100% size Level-0 Level-0 / compression ratio level-0>level-1 = Level-1 size 2000GB/10 = 200GB 2x Level-0 + 3x Level-1 = space to reserve for the archive volume 2x 2000GB + 3x 200GB = 4000GB + 600GB = 4600GB

## Additional Notes

The archive volume will not increase or shrink on his own. When extending this volume please make sure, that no writing process should run in the meantime.
