---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to deploy Exasol in Hetzner datacenter"
summary: "This article describes the deployment of Exasol in data centers and on machines of the German company Hetzner. Hetzner is comparable to AWS or Azure, but the quality of the..."
---
# How to deploy Exasol in Hetzner datacenter

## Overview

This article describes the deployment of Exasol in data centers and on machines of the German company Hetzner. Hetzner is comparable to AWS or Azure, but the quality of the interfaces provided is poor, making deployment a challenge.

## Explanation

In my attempts to install Exasol on dedicated Hetzner machines, I have identified three issues that are critical to the success of the installation:

1. Using a "Raritan" console instead of "KVM" console.
2. Burning the ISO image to a USB stick.
3. Requesting the longest possible console sessions

## Recommendation

### Using a "Raritan" console instead of "KVM" console.

By default, a KVM console is provided by Hetzner. Via this console, it should be possible to mount an SMB share via which the ISO can be mounted. The problem with this procedure is: it does not work. Also the Hetzner support was not able to localize the problem. In addition, parts of the keyboard were not available when operating the KVM console, which made it impossible to work sensibly.

It is therefore recommended to explicitly request the Raritan console. No problems occurred here in terms of operation.

### Burning the ISO image to a USB stick.

Instead of including an SMB share, the Raritan console supports direct virtual mounting of ISOs from the local client. Unfortunately, this does not work either. Although a 50MBit/s upload was available and the machine at Hetzner had 1Gbit/s symmetrical bandwidth no higher transfer speed than 10Mbit/s could be achieved. This leads to a transfer time of >1h for our 4.1GB image. This in turn leads to interrupted downloads due to terminated sessions (see next point), connection aborts and corrupt images.

We did not manage to upload a bootable image in five attempts. It is therefore recommended to send the ISO to Hetzner Support including detailed instructions on how to burn the ISO to a USB stick so that it is bootable.

These are the steps required to make a bootable flash drive:

1. Convert image: isohybrid EXASuite-x.y.z-RELEASE.iso
2. Copy image to USB stick: dd if=EXASuite-x.y.z-RELEASE.iso of=/dev/sda (whereby /dev/sda is the USB device in this case)
3. Boot from USB and at installation time use the option install ks=hd:sda:/install.cfg method=hd:sda:/ (whereby /dev/sda is the device name under which the USB device has been detected)

### Requesting the longest possible console sessions

We have painfully learned that the duration of the installation cannot be estimated in advance. If the installation process is unnecessarily prolonged due to disconnections, this leads to further frustration on the part of the customer and the Exasol employee. It is therefore recommended to request a sufficiently long session window (>5 hours) when applying for the Raritan console.

## Additional References

Hetzner Website: <https://www.hetzner.com/>

KVM Console Docs: <https://docs.hetzner.com/de/robot/dedicated-server/maintainance/kvm-console/>

Raritan Console Docs: <https://docs.hetzner.com/de/robot/dedicated-server/maintainance/kvm-console/#anmelden---raritan-kvm>
