---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL - Passwort Reset Exaoperation"
summary: "The password for an EXAoperation user needs to be reset but there is no other user available."
---
# INTERNAL - Passwort Reset Exaoperation

## Overview

The password for an EXAoperation user needs to be reset but there is no other user available.

## Prerequisites

Root access to the system.

## Execution

## .) Login as root to license server via SSH

## .) Stop EXAoperation

```
cosrm -a {Partition-ID of EXAoperation}
```

## .) Start "exaopctl shell" command and issue the following commands to set a new password for user admin

```
root[u'cluster1'][u'users'][u'0'].setPassword("mynewpassword")
import transaction
transaction.commit()
```

## .) Restart EXAoperation

```
cosexec --single-instance --auto-restart --auto-add -- $COS_DIRECTORY/libexec/appserverd
```
