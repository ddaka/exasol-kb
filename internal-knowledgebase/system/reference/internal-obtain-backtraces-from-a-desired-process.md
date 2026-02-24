---
tool_name: cos
doc_type: reference
category: system
title: "INTERNAL - Obtain backtraces from a desired process"
summary: "Obtain backtraces from the desired process."
---
# INTERNAL - Obtain backtraces from a desired process

## Overview

Obtain backtraces from the desired process.

One-time backtrace collection could be done via standard `get_support_info` / `exasupport` tools.

Approaches below are necessary when one has more strict requirements than could be achieved by the tools.

## Explanation

### All SQL Processes

```shell
while true; do cosexec -rtn 18,19,20,21 bash -c 'pgrep exasql | xargs obtain_backtraces >/tmp/bbtraces.$(date +"%Y%m%d_%H%M")'; sleep 60; done;
```

This shell command takes backtraces of SQL processes (`exasql`) every 60 seconds for the particular set of nodes (in this case `n18,n19,n20,n21`).

### All processes belonging to a particular running session

```shell
psh 'pgrep -f "current_process_nr=36$" | xargs obtain_backtraces >/tmp/backtraces_1803549219144073255.$(hostname -s).$(date +"%Y%m%d_%H%M")'
```

Here we beforehand found in SQL log the number of SQL process executing the session currently (#36) and are dumping backtraces on each node to a file with name `backtraces_<hard-coded session ID>.<node name>.<timestamp>`.

Later on we likely need to copy all these files to the license node, to compress and copy outside the cluster.

```shell
# Create folder for backtraces on the license node
mkdir backtraces_1803549219144073255

# Check what is available
psh ls -l '/tmp/backtraces_1803549219144073255.*.*'

# Copy
psh 'find /tmp | grep backtraces_1803549219144073255 | xargs -I {} scp {} n10:/root/backtraces_1803549219144073255/'
```

When there are many nodes to process, connection to/from some of them might fail like

```text
...
0012: lost connection
0020: lost connection
...
```

In that case one can repeat the last command, but for specific nodes. It can be achieved by executing the same command not via `psh`, but via `cosexec`:

```shell
cosexec -rtn 12,20 bash -c 'the command executed by the psh call above'
```

Here `12,20` means doing it for nodes n12 and n20.

## Additional References

* [Backtrace Information, version 8](https://docs.exasol.com/db/latest/administration/on-premise/support/backtrace_information.htm)
* [Backtrace Information, version 7.1](https://docs.exasol.com/db/7.1/administration/on-premise/support/backtrace_information.htm)
