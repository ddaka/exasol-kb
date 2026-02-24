---
tool_name: c4
doc_type: troubleshoot
category: system
title: "How to add a local file system for backups"
summary: "Local file system mounts can be used if tools like Tivoli TSM are used to store Exasol backups on tape drives. These folders are then used as remote archive volumes using file://..."
---
# How to add a local file system for backups

Local file system mounts can be used if tools like Tivoli TSM are used to store Exasol backups on tape drives. These folders are then used as remote archive volumes using file:// URL.

## Folder permission

Ensure exadefusr is allowed to create folders inside the mount point. You can test this from inside COS with the below commands. If one of the files or folders cannot be read (e.g., lost+found usually belongs to root) backup creation will fail.

```shell
c4 connect -t1/cos -n 11
psh
su - exadefusr
cd /YOURmountPOINT
mkdir somefoldername
```

Additional tests can be done with sdfs-command.

```shell
c4 connect -t1/cos -n 11
touch test.file
sdfs add <REMOTE_VOLUME> - test.file
```

Check output for errors.

## Add local file system mount after the intial setup

The main user-facing interface to adding custom mounts is "CCC_PLAY_MOUNTS". New mounts can be added to an existing system by adding them to ".play.mounts" field of "$HOME/.ccc/ccc/etc/c4.yaml" file (“$HOME” of the exasol user). E.g.:

```yaml
play:
        mounts: ["/mnt/test_outside:/mnt/test_inside"]
```

After you do that, you'll need to restart both the DB and the container with "systemctl --user restart c4_cloud_command". This will need to be done on every node that needs these mount points to be mounted.
The "c4.yaml" file is kept untouched during updates, so this change will persist after future updates.
