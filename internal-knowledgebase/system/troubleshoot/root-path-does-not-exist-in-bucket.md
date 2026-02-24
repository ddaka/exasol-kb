---
tool_name: cos
doc_type: troubleshoot
category: system
title: "BucketFS File not synced"
summary: "In some cases, a file in BucketFS may not syncronize properly between nodes. The exact cause and circumstances surrounding this is unclear."
---
# BucketFS File not synced

## Description

In some cases, a file in BucketFS may not syncronize properly between nodes. The exact cause and circumstances surrounding this is unclear.

The user may report that they uploaded a file to BucketFS, but a UDF cannot find it. Similarly, if the file in question is a script language container, they may report the following error message:

`BucketFS: root path 'EXAClusterOS/ScriptLanguages-standard-EXASOL-7.1.0-slc-v4.0.0-CM4RWW6R/'' does not exist in bucket 'default' of bucketfs 'bfsdefault' `

The name of the file shown in the error message varies depending on the exact file name.

## Diagnosis

You can verify if the file is not syncronized on all nodes by running the following command on one of the data nodes:

`psh ls /path/to/bucketfs/<bucketfsname>/<bucket>/<file>`

For example, if the file `test` was not syncronized, the output would be
```
psh ls /exa/data/bucketfs/bfsdefault/default/test
0011: test
0012: ls: 0012: cannot access /exa/data/bucketfs/bfsdefault/default/test: No such file or directory
```
## How to fix

Use the `cos_sync_files` command to syncronize the files on all nodes:

```
cos_sync_files /exa/data/bucketfs/bfsdefault/default/test
```

Note: If the file is a large tar.gz, it may take several minutes for the new file to actually be available to the database because the file must be unpacked on the nodes.

## Additional References
* [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm)


