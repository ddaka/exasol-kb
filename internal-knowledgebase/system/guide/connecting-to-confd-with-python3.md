---
tool_name: confd_client
doc_type: guide
category: system
title: "Connect to ConfD with Python 3 (XML-RPC)"
summary: "Use Python 3 and xmlrpc.client to authenticate, list available ConfD jobs, and execute administrative jobs on legacy EXAoperation/NGA deployments."
---

# Connect to ConfD with Python 3 (XML-RPC)

## Overview

This guide shows how to connect to ConfD over XML-RPC from Python 3, discover available jobs, and run basic administrative calls.

## Prerequisites

- Python 3 is available.
- `xmlrpc.client` is available (Python standard library).
- `requests` is installed.
- A running legacy EXAoperation/NGA deployment.
- Valid ConfD credentials.

## Procedure

### 1. (Optional) Set root password for legacy environments

If the `root` password is not configured for ConfD authentication in the deployment, set it first:

```bash
exaconf passwd-user --name root --passwd '<strong-password>'
```

Apply according to your environment runbook (service restart or reboot where required).

### 2. Open a Python session and create an XML-RPC connection

```python
import requests
import ssl
from xmlrpc.client import ServerProxy

username = "root"
password = "<password>"
xmlrpc_port = 443

# Query any cluster node to resolve the current master node IP
master_ip = requests.get(
    "https://<any-node-ip>:443/master",
    verify=False,
    timeout=10,
).content.decode("utf-8").strip()

connection_string = f"https://{username}:{password}@{master_ip}:{xmlrpc_port}"
ssl_context = ssl._create_unverified_context()
conn = ServerProxy(connection_string, context=ssl_context, allow_none=True)
```

### 3. List available jobs

```python
conn.job_list()
```

Pretty-print the result if needed:

```python
import pprint
for job in conn.job_list():
    pprint.pprint(job)
```

### 4. Execute a ConfD job

```python
result = conn.job_exec("node_list")
result["result_output"]
```

Access nested fields from the response dictionary:

```python
result["result_output"]["11"]["disks"]
```

## Examples

### Check database state

```python
conn.job_exec("db_state", {"params": {"db_name": "DB1"}})["result_output"]
```

### Retrieve database information

```python
db_info = conn.job_exec("db_info", {"params": {"db_name": "DB1"}})
db_info["result_output"]
```

## Security Notes

- Avoid hardcoded credentials in scripts.
- Replace `verify=False` with certificate validation in production.
- Prefer dedicated service users and least-privilege permissions over shared root credentials.

## References

- [Using XML-RPC to manage Docker clusters](https://exasol.my.site.com/s/article/Using-XML-RPC-to-manage-Docker-clusters)
- [ConfD XML-RPC](https://exasol.atlassian.net/wiki/spaces/RD/pages/12159574/ConfD+XMLRPC)
