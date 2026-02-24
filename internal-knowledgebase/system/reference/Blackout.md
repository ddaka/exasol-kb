---
tool_name: cos
doc_type: reference
category: system
title: "Blackout"
summary: "There was a blackout in the data center and the system needs to be started again."
---
# Blackout

## Overview

There was a blackout in the data center and the system needs to be started again.

## Explanation

First check which nodes are available and which are not.
- power on the license server
- check if there is any other nodes currently running - if so, stop COS there
- start all data nodes
- check data nodes network
- start EXAStorage
- check disks, volumes
- start database
- ask the customer to connect


