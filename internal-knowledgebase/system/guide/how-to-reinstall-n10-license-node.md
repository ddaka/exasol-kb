---
tool_name: cos
doc_type: guide
category: system
title: "How to reinstall n10 (license node) after hardware replacement"
summary: "Runbook for rebuilding the n10 license node with matching cluster configuration and rejoining it safely after hardware replacement."
---
# How to reinstall n10 (license node) after hardware replacement

## Purpose

Rebuild the license node (`n10`) after hardware replacement and rejoin it to an existing cluster.

## Scope

Legacy/on-prem style environments where license node topology applies.

## Critical prerequisites

- Reinstall with the exact same Exasuite ISO version and OS patch level as data nodes.
- Collect reference configuration from a healthy data node before starting.
- Plan maintenance window.

## Procedure

1. On a data node, record private network and n10 mapping:
   - `cat /etc/cos/private_network.cfg`
   - `grep n10 /etc/hosts`
2. Reinstall `n10` with matching ISO/patch.
3. Connect to `n10` (port 20 namespace) and stop COS:
   - `systemctl stop cos`
4. Verify on `n10`:
   - `/etc/cos/private_network.cfg` matches data nodes.
   - `/etc/cos.conf` contains required matching configuration (edit manually, do not blindly overwrite).
5. Copy required cluster artifacts from a data node:

```shell
scp -P20 -p /etc/cos/cored_random root@n10:/etc/cos/cored_random
scp -P20 -pr ~/.ssh root@n10:~
```

6. Enter host context (port 22) and synchronize storage config:

```shell
ssh localhost -p22
cos_sync_files $COS_DIRECTORY/etc/cos_storage_real.conf
```

7. Start COS on `n10`:

```shell
systemctl start cos
```

8. Monitor logs until node rejoin is confirmed (for example message indicating node `10` added back to EXAStorage cluster).
9. Move EXAoperation role back to license node if required by environment standard.

## Notes

- Exasol v7 is end-of-life; use at your own risk and follow internal approval paths.
- `cos_sync_files` command details are maintained in canonical doc:
  - `documents/cos/cos-sync-files.md`

## References

- <https://docs.exasol.com/db/7.1/administration/on-premise/installation/install_hw.htm>
- <https://docs.exasol.com/db/7.1/administration/on-premise/admin_interface/exaoperation.htm>
- <https://downloads.exasol.com/legacy-releases>


