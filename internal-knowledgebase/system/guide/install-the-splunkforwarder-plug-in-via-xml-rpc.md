---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Install the Splunkforwarder plugin via XML-RPC"
summary: "Legacy EXAoperation XML-RPC workflow for installing and configuring the Splunkforwarder administration plugin."
---
# Install the Splunkforwarder plugin via XML-RPC

## Purpose

Install and configure the Splunkforwarder plugin through EXAoperation XML-RPC calls.

## Scope

Legacy workflow for environments still using EXAoperation plugin RPC APIs.

## 1) Upload plugin package in EXAoperation

Upload package (example):

- `Plugin.Administration.splunkforwarder-7.2.4-1.0.2-2019-08-22.pkg`

Path in UI:

- `Configuration -> Software -> Versions -> Browse -> Submit`

## 2) Connect to XML-RPC endpoint (Python example)

```python
import ssl
import pprint
import xmlrpc.client

proxy = xmlrpc.client.ServerProxy(
    "http://<user>:<password>@<license-server>/<cluster>",
    context=ssl._create_unverified_context(),
    allow_none=True,
)
```

## 3) Inspect available plugin functions

```python
pprint.pprint(proxy.showPluginFunctions("Administration.splunkforwarder-7.2.4-1.0.2"))
```

## 4) Install plugin on target node

```python
status, ret = proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "INSTALL")
print(status, ret)
```

Expected success return code: `0`.

## 5) Check status

```python
proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "STATUS")
```

## 6) Configure forward server

```python
proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "ADD_SERVER", "10.70.0.186:9997")
```

## 7) Upload `inputs.conf`

```python
config = open("/root/inputs.conf").read()
proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "PUT_CONFIG", config)
```

## 8) Verify config and restart

```python
proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "GET_CONFIG")
proxy.callPlugin("Administration.splunkforwarder-7.2.4-1.0.2", "n10", "RESTART")
```

Repeat installation/configuration for all required nodes.

## De-duplication note

Canonical plugin command references (non-EXAoperation-RPC path) are maintained in:

- `documents/cos/confd-system-and-infrastructure.md`
- `documents/cos/confd-overview.md`


