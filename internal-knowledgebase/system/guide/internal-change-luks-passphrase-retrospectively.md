---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - Change LUKS passphrase retrospectively"
summary: "Internal procedure to rotate LUKS disk passphrase by adding a new key and removing the old key across cluster nodes."
---
# Internal - Change LUKS passphrase retrospectively

## Purpose

Rotate LUKS passphrase without full cluster downtime by replacing key slot material on encrypted devices.

## Safety notes

- Test first on reserve node.
- Keep rollback passphrase available until validation is complete.
- Handle plaintext key files as sensitive data and remove securely after use.

## Procedure

1. SSH to management node.
2. Create files with old and new passphrases (newline required):

```shell
echo "<OLD_PASSPHRASE>" > formerkey.file
echo "<NEW_PASSPHRASE>" > key.file
```

3. Use automation script (example):

```bash
#!/bin/bash
for i in $(ls /usr/opt/EXASuite-5/EXAClusterOS-5.0.11/var/exaoperation/cluster1/nodes); do
  echo "==== Current node: $i"
  DEVICES="$(awk '/luksOpen/ { print $6 }' /usr/opt/EXASuite-5/EXAClusterOS-5.0.11/var/exaoperation/cluster1/nodes/$i/hddmount_gpt.sh)"
  scp key.file $i:.
  scp formerkey.file $i:.
  for dev in $DEVICES; do
    ssh $i cryptsetup luksAddKey $dev key.file --key-file formerkey.file
    ssh $i cryptsetup luksRemoveKey $dev --key-file formerkey.file
  done
  ssh $i "rm -f key.file formerkey.file"
done
```

4. Run script from `tmux` session on management node.
5. Update disk password in EXAoperation (`Access Management`).
6. Restart EXAoperation.
7. Securely remove local key files:

```shell
rm -f key.file formerkey.file
```

## Validation

- Confirm encrypted devices unlock successfully on all nodes.
- Confirm node/database/storage operations remain healthy.
- Confirm EXAoperation stored disk password is updated.


