---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Internal - Workaround for Azure Blob Storage Issues on Exasol 6.0.7"
summary: "If the Azure Blob Storage is not accessible on Exasol 6.0.7, you should do the following:"
---
# Internal - Workaround for Azure Blob Storage Issues on Exasol 6.0.7

If the Azure Blob Storage is not accessible on Exasol 6.0.7, you should do the following:

1. Check the owner of the ***/home/exasolution*** directory, if it's not ***exasolution*** then:

```
$ cosexec -art chown -R exasolution:exasolution /home/exasolution
```

2. Check the owner of the ***/etc/cos/sdfs_remote_volumes.cfg*** file, if it's not ***exasolution*** then:

```
$ cosexec -art chown -R exasolution:exasolution /etc/cos/sdfs_remote_volumes.cfg
```
3. Check the **/etc/passwd** file and change the ***exasolution*** user's to 1000 if it is something else.

```
$ sudo sed -i "s/^exasolution:x:500:/exasolution:x:1000:/" /etc/passwd
```
