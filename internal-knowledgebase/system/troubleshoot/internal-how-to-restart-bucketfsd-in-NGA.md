---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to restart bucketfsd in pre-NGA"
summary: "In pre-NGA, the 'bucketfsd' subsystem is known to be unreliable and sometimes it needs a 'good old' restart in order to sync the bucket contents. There are 3 ways to do that:"
---
# How to restart bucketfsd in pre-NGA

## Problem

In pre-NGA, the 'bucketfsd' subsystem is known to be unreliable and sometimes it needs a 'good old' restart in order to sync the bucket contents.  There are 3 ways to do that:

### Solution #1
**(RECOMMENDED)** - Restart bucketfsd, using 'cosrst'
```
cosrst {PID} {NID}
```
### Solution #2
Restart bucketfsd, using 'coskillall'. This method is not the best, but it will do the trick, having the same PID, FLAGS, and COMMAND name.
```
coskillall -9 bucketfsd-bfsdefault
```
### Solution #3
Restart bucketfsd, removing the PID. This method will start bucketfsd with the correct UID/GID, name, and other parameters.
```
cosrm -a {PID}
exaconf commit
```

- If further troubleshooting is needed check the logs in `/exa/logs/cored`
