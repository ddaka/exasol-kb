---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Failsafety in the Public Clouds"
summary: "In all Public Clouds, Exasol has the so-called CloudUI Plugin and CloudUI Backend installed (verify in EXAoperation Software packages page). Those plugins add new failsafety..."
---
# Failsafety in the Public Clouds

### Overview

In all Public Clouds, Exasol has the so-called CloudUI Plugin and CloudUI Backend installed (verify in EXAoperation Software packages page). Those plugins add new failsafety functionalities. These failsafety features are supported in AWS, Azure and GCP. The standard failsafety mechanisms are also available in the Public Clouds.

1. Cold-standby reserve node
2. Recover "failed" node(s) - failed has multiple meaning in that context those meanings will be described in this article

***Pros of the Public Clouds:***

1. Flexibility: on-demand resource allocation
2. Freedom of choice: change resource types any time
3. Pricing: every resource has a price "tag" it is pretty easy to predict costs
4. Availability: SLAs available for most of the services
5. APIs for every infrastructure component

***Cons of the Public Clouds:***

1. You pay for a VM that is running it doesn't matter if it is using 1% CPU or 100% CPU
2. You pay for block storage as soon as it is allocated it doesn't matter if you're using it or not
3. Predictable costs and cloud pricing calculators cause customers to start saving costs
4. New requirements towards our software in regards of cost-savings and on-demand resource allocation

Failsafety can be implemented in many ways this article describes the failsafety for Public Clouds. Public Clouds setups most of the time should be as cost-optimised as possible and Public Clouds already offer high-availability for their e.g. block storage backends that is why EXAStorage redundancy is kept at a low level of 1. From a cost perspective it also makes sense to not have a hot-standby node running all the time. That is why we needed new methods to recover failed nodes.

### Cold-standby node

The first request was to convert the hot-standby node into a on-demand cold-standby - that means it will be powered-on only in case of a node failure. Costs will only occur for the storage volumes and if the VM is online. A cold standby only makes sense if the EXAStorage DATA volume of the database is using a redundancy of 2 otherwise data is missing and cannot be restored.

### Setup

The setup is quite simple just install the cluster with a standby node and shut it down once the database is up and running - and you're done.

### Failsafety process

These are the steps in case of a node failure:

1. Try to recover the failed node by rebooting it
2. Failed node cannot be recovered by rebooting it
3. Power On the cold-standby node
4. Node will boot and also install any missing software updates
5. Node will join the COS partitions
6. Node will join EXAStorage
7. Node will be pushed in as active database node
8. Database will be started

### Duration

The duration depends on:

1. if the specified instance type is available in that moment - might be that the resource pool of that instance type is exhausted
2. the node was offline for a long time and is missing the latest software packages
3. disks got corrupted and the node cannot boot - disks need to be fixed first
4. Step 1 (recovering the failed node) takes very long if the node is stuck (15 minutes or more depends on the cloud platform)

### No standby node

The next step was to go even further, so no standby node at all. That means the system tries to recover the failed node it will not be replaced by a standby node. In this setup it even makes sense to reduce the EXAStorage DATA volume redundancy to 1. Why? Public clouds block storage device are highly available and we don't use a spare node. Let's put it like this: "We trust in the integrity and availability of the platform". I will not put any numbers here as they can be found in the internet.

### Setup

None - the CloudUI plugin takes care of it.

### Failsafety process

In all three scenarios the CloudUI plugin tries to recover the node by either powering it on or rebooting it, once it is online again it tries to start the database.

### Scenarios

1. an active database node is rebooted
2. an active database node is offline
3. an active database node is unavailable e.g. network link goes down

### The CloudUI Plugin

The CloudUI plugin is a set of Python scripts that talk to the cloud APIs and can do specific tasks like rebooting a node, starting a node and stopping a node. The CloudUI plugin is installed on the mgmt. node only - yes this is a single point of failure and we should think about making it high available.
