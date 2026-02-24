---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How-to resize d02_data"
summary: "In order to do a resize, you have to restart the affected nodes."
---
# How-to resize d02_data

 In order to do a resize, you have to restart the affected nodes.

 Ensure d02_data is the last partition of the disk

 Ensure d02_data is set to "Rest" in EXAoperation nodes disk configuration, if not change Install/Active flags temporary

In order to do a resize, you have to restart the affected nodes.

The way to do the resize differs, whether you use a VM or bare metal.

## How to resize d02_data on VM

## Step 1

Resize your hard disk in your VM

## Step 2

Run "cos-dmesg" on the affected node in order to check if the kernel recognizes the new size of the hard disk

## Step 3

Resize your partition

* start "parted"
* if this warning shows up:    Warning: Not all of the space available to /dev/vda
appears to be used, you can fix the GPT to use all
of the space (an extra 104857600 blocks) or
continue with the current setting?

    -> type "Fix"
* type "resizepart [affected_partition]"
* if this warning shows up: Partition /dev/vda4 is being used. Are you
sure you want to continue?

 -> type "Yes"
* type "100%" for End
* if this warning shows up: Error: Error informing the kernel about
modifications to partition /dev/vda4 – Invalid
argument. This means Linux won't know about any
changes you made to /dev/vda4 until you reboot –
so you shouldn't mount it or use it in any way
before rebooting.

 -> type "Ignore"
* check if the partition is resized with "print"

## Step 4

Shutdown DB

## Step 5

Shutdown Cluster Services

## Step 6

Reboot affected Nodes

## Step 7

Run "resize2fs [partition]" on the node

## Step 8

Check the size of the partition in EXAOperation (Nodes -> click node -> click "Disks")

## Step 9

Start Cluster Services

## Step 10

Start DB

## How to resize d02_data on bare metal

## Step 1

Shutdown DB

## Step 2

Shutdown Cluster Services

## Step 3

Resize the partition on the node (see steps above)

## Step 4

Reboot affected Nodes

## Step 5

Run "resize2fs [partition]" on the node

## Step 6

Check the size of the partition in EXAOperation (Nodes -> click node -> click "Disks")

## Step 7

Start Cluster Services

## Step 8

Start DB
