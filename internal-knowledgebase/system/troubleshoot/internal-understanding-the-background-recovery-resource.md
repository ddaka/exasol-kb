---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL - Understanding the background recovery resource limitation mechanism"
summary: "The EXAStorage background recovery limit is used to limit the throughput of the background data restoration, which is automatically started after a failed node or disk has been..."
---
# INTERNAL - Understanding the background recovery resource limitation mechanism

## Overview

The EXAStorage background recovery limit is used to limit the throughput of the background data restoration, which is automatically started after a failed node or disk has been recovered. This leaves more I/O and network resources for usage by the DB.

## Explanation

The limit can be changed at any time, as long as EXAStorage is running. When the limit is changed (or the service is restarted), EXAStorage will start a "calibration phase". During that phase, it adjusts internal parameters until the requested limit is reached (if possible). After that phase, the parameters are only modified if the limit is exceeded (with a few percent tolerance), i. e. the throughput will never be significantly higher than the limit but may be significantly lower if the DB uses a lot of resources. The calibration phase can be manually restarted by using the "Force recalibration" button (or changing the limit).

The limit can be set for each node individually (in MiB/s). Initially, the service will automatically select a value that depends on the speed of the network interface:

1 GBit interface : 75 MiB/s
10 Gbit interface: 300 MiB/s

If the current values have been automatically selected, the label "auto" will be displayed after the limit itself (see screenshots). In order to return to the default ("auto") values, the limit has to be set to 0. In order to have the max. possible throughput (e. g. when waiting for a restoration without a running DB), set the limit to a very high value (e. g. 200 MiB/s for 1GBit systems).

![](images/Limit_1.png)

![](images/Limit_2.png)
