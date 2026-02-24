---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - Host Health Check"
summary: "With instance status monitoring, you can quickly determine whether Amazon EC2 has detected any problems that might prevent your instances from running applications. Amazon EC2..."
---
# SaaS - Host Health Check

## Overview

With instance status monitoring, you can quickly determine whether Amazon EC2 has detected any problems that might prevent your instances from running applications. Amazon EC2 performs automated checks on every running EC2 instance to identify hardware and software issues.

Status checks are performed every minute, returning a pass or a fail status. If all checks pass, the overall status of the instance is **OK**. If one or more checks fail, the overall status is **impaired**. Status checks are built into Amazon EC2, so they cannot be disabled or deleted

## Type of Status checks

There are two types of status checks: system status checks and instance status checks.

```sql
System Status checks

The following are examples of problems that can cause system status checks to fail:
    Loss of network connectivity
    Loss of system power
    Software issues on the physical host
    Hardware issues on the physical host that impact network reachability

Instance status checks

The following are examples of problems that can cause instance status checks to fail:
    Failed system status check
    Incorrect networking or startup configuration
    Exhausted memory
    Corrupted file system
    Incompatible kernel
```

## Procedure

If your instance is in **impaired** state, you must try to stabilize the instance by following the steps below:

* ***Stop** Instance*
* ***Start** Instance*
* *Check Disk Status -> [link](saas-disk-status.md)*
* *Check Disk Usage -> [link](saas-disk-usage-root-and-home-disks.md)*

`Note: Please DO NOT use the reboot feature, instead use Stop and Start instance. Doing so will place instance in different hardware and might fix the issue. Using reboot feature will not place instance in different hardware.`

However, if the instance can not be stabilized using above steps, it is still in **impaired** state, and it has been in that state for more than 20 minutes, please contact the **Production Engineering Team**.

In the meantime, you can create an AWS support ticket and provide all needed information for AWS support team to troubleshoot the instance. In order to do so, please follow the steps in this [link](https://docs.aws.amazon.com/awssupport/latest/user/case-management.html).

## Report instance status

You can provide feedback if you are having problems with an instance whose status is not shown as impaired, or if you want to send AWS additional details about the problems you are experiencing with an impaired instance.

### Procedure

1. Open the Amazon EC2 console
2. In the navigation pane, choose **Instances**.
3. Select the instance, choose the **Status Checks** tab, choose **Actions** (the second **Actions** menu in the bottom half of the page), and then choose **Report instance status**.
4. Complete the **Report instance status** form, and then choose **Submit**.
