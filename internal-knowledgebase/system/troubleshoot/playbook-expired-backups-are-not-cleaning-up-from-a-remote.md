---
tool_name: cos
doc_type: troubleshoot
category: system
title: "PLAYBOOK: Expired backups are not cleaning up from a remote archive"
summary: "The expired backups are not cleaning up from a remote archive (AWS S3, Azure Blob Storage, Google Cloud Storage, etc...) after a backup has run."
---
# PLAYBOOK: Expired backups are not cleaning up from a remote archive

## Overview

The expired backups are not cleaning up from a remote archive (AWS S3, Azure Blob Storage, Google Cloud Storage, etc...) after a backup has run.

## Diagnosis

In EXAoperation (**DB Name -> Exasolution Instance -> Backups -> Show expired Backups**), you are able to see a backup list of some expired backups like in the image below:

![Backup list in EXAoperation](images/EXASolution_BackupList.png)

If today's date is far after the date of the "expired" backups, then you are facing this issue.

## Explanation

You can store your backups in a remote location. Exasol supports FTP, SMB, Amazon S3, WebHDFS, Azure Blob Storage and Google Cloud Storage as a remote archive. Some of the benefits of the remote archive volume are:

* It is beneficial when it comes to fail-safety. If your complete cluster crashes because of any reason, the remote backup is still safe and you can use it to restore your cluster from the remote backup.
* It supports [**Blocking Restore**](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore/restore_database.htm).

The old backups are not getting removed after they get expired and therefore space is not claimed back to the volume, so new backups may also fail. The reason is perhaps hitting a bug while creating the backup on where an expiration date has not being set or simply the backup expiration was not defined originally.

## Recommendation

Please have a look at the suggested actions:

* Please check whether "*cleanvolume"* parameter has been set on the remote archive volume in Exaoperation (**ExaStorage -> Remote Archive -> Remote Archive Volume Information -> Options** ).
* Try to set an expire timestamp of the specified file(s) of a backup from the remote volume (or only mark as deleted if there is enough space) by using the "*sdfs*" command like:

```shell
sdfs expire {volume} {timestamp|-} {file1} ...
```

where {volume} is the volume ID for the remote archive volume, i.e. r0001.

* Try to remove the expired file(s) from the volume (or only mark as deleted if there is enough space) by using the "*sdfs*" command:

```shell
sdfs remove {volume} [-r] {file1} ...
```

* If otherwise, you have hit the Bug described [here](https://exasol.my.site.com/s/article/Changelog-content-10530?language=en_US) (this is for GCP) you may need to update your database to the latest Exasol version.

## Additional References

For details or more information, check out the links below:

* [Create Remote Archive Volume](https://docs.exasol.com/db/latest/administration/on-premise/manage_storage/create_remote_archive_volume.htm)
* [CHANGELOG: Expired backups on Google Cloud Storage are not removed](https://exasol.my.site.com/s/article/Changelog-content-10530?language=en_US)
