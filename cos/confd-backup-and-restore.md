---
tool_name: confd_client
doc_type: reference
category: Backup and Restore
subcommands: db_backup_start, db_backup_list, db_backup_abort, db_backup_progress, db_backups_delete, db_backup_add_schedule, db_backup_modify_schedule, db_backup_remove_schedule, db_backup_change_expiration, db_backup_delete_unusable, db_restore, db_snapshot_backup_start, db_snapshot_backup_list, db_snapshot_backup_add_schedule, db_snapshot_backup_modify_schedule, db_snapshot_backup_remove_schedule, db_snapshot_backups_delete
---

# confd_client — Backup and Restore

## Overview

Commands for managing database backups, restore operations, snapshot backups, and backup schedules.

All commands run inside the COS namespace (SSH port 20002). The database must be running for backup operations and offline for restore operations.

## db_backup_start

This job starts a backup of the given database to the given archive volume
with given level and expiration time. The database must be running in order
to write a backup.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_volume_id` (int, required): ID of the archive volume which will store the backup. The archive volume can be either local or remote. This parameter may be substituted by backup_volume_name.
- `db_name` (str, required): Name of an existing database to be backed up.
- `level` (int, required): The backup level (0,1,2,3,etc). Level 0 means a full backup, levels 1-9 are incremental backups.
- `backup_volume_name` (str, optional): Name of the archive volume which will store the backup. The archive volume can be either local or remote. If both backup_volume_id and backup_volume_name are specified, backup_volume_id is used.
- `expire` (str, optional): 'The expiration time for the backup using the format "#m #w #d #h" (ie, "1w 3d"). If no value is specified, the backup will not expire.'

**Examples**:

```bash
confd_client db_backup_start {backup_volume_name: ArchiveVolume1, db_name: exa_db, expire: 1w, level: 0}
```

## db_backup_list

This job returns a list of available backups for the given database.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database.
- `show_foreign` (bool, optional): Shows backups for databases other than the one specified in db_name.

**Examples**:

```bash
confd_client db_backup_list {db_name: exa_db}
```

## db_backup_abort

This job aborts a running database backup.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database with a running backup.

**Examples**:

```bash
confd_client db_backup_abort {db_name: exa_db}
```

## db_backup_progress

This job returns the progress of the currently-running backup for a given
database.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of the database where a backup is running.

**Examples**:

```bash
confd_client db_backup_progress {db_name: exa_db}
```

## db_backups_delete

This job deletes the specified backup(s).

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_list` (list, required): List of IDs of backups to be deleted. A backup ID has the format "<volume_id> <path_to_backup> <db_name>", for example "1 exa_db/id_1/level_0/node_0/backup_201811061154 exa_db". This can be obtained f
- `db_name` (str, required): Name of an existing database.

**Examples**:

```bash
confd_client db_backups_delete {backup_list: [1 exa_db/id_1/level_0/node_0/backup_201811131114 exa_db], db_name: exa_db}
confd_client db_backups_delete {backup_list: [1 exa_db/id_1/level_0/node_0/backup_201811131114 exa_db, 1 DB1/id_2/level_0/node_0/backup_201811201114
```

## db_backup_add_schedule

This job adds a backup schedule to the given storage volume with the given
level and expiration time.

    expire: 1w 3d, hour: 0, level: 0, month: '*', weekday: 0}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): The name of the backup schedule.
- `backup_volume_id` (int, required): The ID of the archive volume which will store the backups. backup_volume_id can be substituted by backup_volume_name.
- `db_name` (str, required): The name of an existing database.
- `level` (int, required): The backup level (0,1,2,3,etc). Level 0 means a full backup, levels 1-9 are incremental backups.
- `backup_volume_name` (str, optional): The name of the archive volume which will store the backups. backup_volume_name can substitute backup_volume_id.
- `day` (str|int, optional): Which day the backup should run. The format is the same as days in cronjob (1 - 31 or *). If not specified, the default value is "*".
- `enabled` (bool, optional): When set to True, the schedule is enabled immediately. Otherwise, the schedule will not run until it is enabled.
- `expire` (str|int, optional): 'The expiration time for the backup using the format "#m #w #d #h" (ie, "1w 3d"). If not specified, the backup will not expire.'
- `hour` (str|int, optional): Which hour the backup should run. The format is the same as hours in cronjob (0 - 23 or *). If not specified, the default value is "*".
- `minute` (str|int, optional): Which minute the backup should run. The format is the same as minutes in cronjob (0 - 59 or *). If not specified, the default value is "0".
- `month` (str|int, optional): Which month the backup should run. The format is the same as months in cronjob (1 - 12 or *). If not specified, the default value is "*".
- `weekday` (str|int, optional): Which weekday the backup should run. The format is the same as weekdays in cronjob (0 - 7 or the english name of the day or *). If not specified, the default value is "*".

**Examples**:

```bash
confd_client db_backup_add_schedule {backup_name: backup1, backup_volume_name: r0001, day: '*', db_name: exa_db, enabled: true,
```

## db_backup_modify_schedule

This job modifies the given backup schedule with new parameters. The
database must be running to run this job.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): The name of the backup schedule to be modified.
- `db_name` (str, required): The name of an existing database.
- `day` (str|int, optional): Which day the backup should run. The format is the same as days in cronjob (1 - 31 or *).
- `enabled` (bool, optional): When set to True, the schedule is enabled immediately. Otherwise, the schedule will not run until it is enabled.
- `expire` (str|int, optional): 'The expiration time for the backup using the format "#m #w #d #h" (ie, "1w 3d"). If not specified, the backup will not expire.'
- `hour` (str|int, optional): Which hour the backup should run. The format is the same as hours in cronjob (0 - 23 or *).
- `minute` (str|int, optional): Which minute the backup should run. The format is the same as minutes in cronjob (0 - 59 or *).
- `month` (str|int, optional): Which month the backup should run. The format is the same as months in cronjob (1 - 12 or *).
- `weekday` (str|int, optional): Which weekday the backup should run. The format is the same as weekdays in cronjob (0 - 7 or the english name of the day or *).

**Examples**:

```bash
confd_client db_backup_modify_schedule {backup_name: backup1, day: '*', db_name: exa_db, hour: '0', minute: '0', month: '*',
confd_client db_backup_modify_schedule {backup_name: backup1, db_name: exa_db, enabled: true}
```

## db_backup_remove_schedule

This job removes the given backup schedule.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): The name of the backup schedule.
- `db_name` (str, required): The name of an existing database.

**Examples**:

```bash
confd_client db_backup_remove_schedule {backup_name: backup1, db_name: exa_db}
```

## db_backup_change_expiration

This job changes the expiration time of the given backup.

      5:00:00'}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_files` (str, required): Prefix of the backup files. The prefix is generally in the format "<db_name>/<id_#>/<level_#>", e.g. exa_db/id_1/level_0).
- `backup_volume_id` (int, required): ID of the archive volume storing the backup. Can be substituted by backup_volume_name.
- `expire_time` (str, required): Expiration time for the backup in the format YYYY-MM-DD hh:mm:ss. If the value is blank, the backup will not expire.
- `backup_volume_name` (str, optional): Name of the archive volume storing the backup. Can substitute backup_volume_id.

**Examples**:

```bash
confd_client db_backup_change_expiration {backup_files: exa_db/id_1/level_0, backup_volume_id: 1, expire_time: '2022-05-14
```

## db_backup_delete_unusable

This job deletes all unusable backups for the given database. An unusable
backup is a backup which cannot be read or used anymore, for example a level
1 backup without a corresponding level 0 backup.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database with unusable backups.

**Examples**:

```bash
confd_client db_backup_delete_unusable {db_name: exa_db}
```

## db_restore

This job restores the given database from the given backup. The backup must
meet certain prerequisites defined in documentation, such as originating
from a database with the same number of nodes. The database must be offline
to perform a restore.

    restore_type: blocking}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_id` (str, required): ID of an existing backup. A backup ID has the format "<volume_id> <path_to_backup> <db_name>", for example "1 exa_db/id_1/level_0/node_0/backup_201811061154 exa_db". This can be obtained from the retu
- `db_name` (str, required): Name of an existing database to be restored.
- `restore_type` (str, required): Type of restore. The value must be either 'blocking', 'nonblocking', or 'virtual access'.

**Examples**:

```bash
confd_client db_restore {backup_id: 1 exa_db/id_1/level_0/node_0/backup_201811131114 exa_db, db_name: exa_db,
```

## db_snapshot_backup_start

This job starts a database snapshot with the given expiration time. The
database must be running to run this job.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database which will have the snapshot.
- `expire` (str, optional): 'The expiration time for the snapshot using the format "#m #w #d #h" (ie, "1w 3d"). If no value is specified, the snapshot will not expire.'

**Examples**:

```bash
confd_client db_snapshot_backup_start {db_name: exa_db, expire: 1w}
```

## db_snapshot_backup_list

This job returns a list of available snapshots for the given database.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database with snapshots.

**Examples**:

```bash
confd_client db_snapshot_backup_list {db_name: exa_db}
```

## db_snapshot_backup_add_schedule

This job adds a database snapshot schedule for the given database. The
database must be configured with a snapshot_sync_volume, which is used for
the schedule.

      3d, hour: '*', minute: '0', month: '*', weekday: '*'}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): Name of backup task.
- `db_name` (str, required): The name of an existing database.
- `day` (str|int, optional): Which day the snapshot should run. The format is the same as days in cronjob (1 - 31 or *). If not specified, the default value is "*".
- `enabled` (bool, optional): When set to True, the schedule is enabled immediately. Otherwise, the schedule will not run until it is enabled.
- `expire` (str|int, optional): 'The expiration time for the snapshot using the format "#m #w #d #h" (ie, "1w 3d"). If not specified, the snapshot will not expire.'
- `hour` (str|int, optional): Which hour the snapshot should run. The format is the same as hours in cronjob (0 - 23 or *). If not specified, the default value is "*".
- `minute` (str|int, optional): Which minute the snapshot should run. The format is the same as minutes in cronjob (0 - 59 or *). If not specified, the default value is "0".
- `month` (str|int, optional): Which month the snapshot should run. The format is the same as months in cronjob (1 - 12 or *). If not specified, the default value is "*".
- `weekday` (str|int, optional): Which weekday the snapshot should run. The format is the same as weekdays in cronjob (0 - 7 or the english name of the day or *). If not specified, the default value is "*".

**Examples**:

```bash
confd_client db_snapshot_backup_add_schedule {backup_name: snapshot1, day: '*', db_name: exa_db, enabled: true, expire: 1w
```

## db_snapshot_backup_modify_schedule

This job modifies the given snapshot schedule with new parameters. The
database must be running to run this job.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): The name of the snapshot schedule to be modified.
- `db_name` (str, required): The name of an existing database.
- `day` (str|int, optional): Which day the snapshot should run. The format is the same as days in cronjob (1 - 31 or *).
- `enabled` (bool, optional): When set to True, the schedule is enabled immediately. Otherwise, the schedule will not run until it is enabled.
- `hour` (str|int, optional): Which hour the snapshot should run. The format is the same as hours in cronjob (0 - 23 or *).
- `minute` (str|int, optional): Which minute the snapshot should run. The format is the same as minutes in cronjob (0 - 59 or *).
- `month` (str|int, optional): Which month the snapshot should run. The format is the same as months in cronjob (1 - 12 or *).
- `weekday` (str|int, optional): Which weekday the snapshot should run. The format is the same as weekdays in cronjob (0 - 7 or the english name of the day or *).

**Examples**:

```bash
confd_client db_snapshot_backup_modify_schedule {backup_name: snapshot1, day: '*', db_name: exa_db, hour: '*', minute: '15', month: '*',
confd_client db_snapshot_backup_modify_schedule {backup_name: snapshot1, db_name: exa_db, enabled: true}
```

## db_snapshot_backup_remove_schedule

This job removes the given snapshot schedule.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backup_name` (str, required): The name of the snapshot schedule.
- `db_name` (str, required): The name of an existing database.

**Examples**:

```bash
confd_client db_snapshot_backup_remove_schedule {backup_name: snapshot1, db_name: exa_db}
```

## db_snapshot_backups_delete

This job deletes the given database snapshots.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database.
- `snapshot_backup_list` (list, required): List of snapshot IDs to be deleted, e.g. "exa_db/id_1/level_0/node_0/backup_202108121809". This can be obtained from the return value of the job db_snapshot_backup_list under the key id.

**Examples**:

```bash
confd_client db_snapshot_backups_delete {backup_list: [exa_db/id_1/level_0/node_0/backup_202108121809], db_name: exa_db}
```
