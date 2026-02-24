---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Exasol Server Hardware Troubleshooting Guide"
summary: "Occasionally, it becomes necessary to troubleshoot hardware issues on Exasol servers, especially when alerts are received for faulty or predicted failures for disk/memory, or when..."
---
# Exasol Server Hardware Troubleshooting Guide

## Introduction

Occasionally, it becomes necessary to troubleshoot hardware issues on Exasol servers, especially when alerts are received for faulty or predicted failures for disk/memory, or when network links are reported as down. This knowledge base article provides step-by-step instructions on how to perform a hardware check on Exasol servers.

## Troubleshooting Faulty or Predicted Disk Failure

1. **Run OpenManage Command:**
   Whenever an alert for a faulty disk/memory is received, execute the following command from n10 of the Exasol cluster:

   ```bash
   psh '/opt/check_omsa/check_openmanage-3.7.12/check_openmanage; echo '' psh
2. **Identify the Specific Node:**
   Once you've confirmed the node with the disk failure, find its serial number using the following command:

   ```bash
   dmidecode -t | grep "Serial"
3. **Alternative Verification via iDRAC:**
   Connect to the server via iDRAC to further verify disk failure or any other hardware issues. To find the iDRAC node's IP address, use the command:

   ```bash
   pspsh 'ipmitool lan print|grep "IP Address"
4. **Replacement Procedure:**
   With the failed disk/memory information, refer to the system support page's KB article named 'Faulty Hardware Replacement' to arrange for a replacement with Dell or the appropriate team responsible for the replacement.

## Troubleshooting Network Link Issues

1. **Check IP Address:**
   Verify if the network link is down by checking the server's IP address.

   ```bash
   ip address
2. **Bonding Configuration Check:**
   If bonding is configured, run the following command to check the status:

   ```bash
   cat /proc/net/bonding/bond1(Replace bond1 with the specific bonding configuration for the cluster)
3. **Check Logs Using dmesg:**
   Run the following command on the node to check for network link issues, or any other hardware related issues:

   ```bash
   dmesg -T

## Conclusion

Following these steps should help you effectively troubleshoot and address hardware issues on Exasol servers.
