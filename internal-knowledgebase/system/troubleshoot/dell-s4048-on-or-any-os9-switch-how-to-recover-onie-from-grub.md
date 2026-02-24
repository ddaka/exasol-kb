---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Dell S4048-ON (or any OS9 switch) - How to recover ONIE from grub rescue"
summary: "When trying to update to OS10 from OS9 via the ONIE shell you can experience a problem with the installation file and it will **brick** the switch. If you reboot from the ONIE..."
---
# Dell S4048-ON (or any OS9 switch) - How to recover ONIE from grub rescue

## Overview

When trying to update to OS10 from OS9 via the ONIE shell you can experience a problem with the installation file and it will **brick** the switch. If you reboot from the ONIE shell and try to do a normal boot, it will succeed, but after a few seconds it will do an **Emergency Reboot**. From the GRUB menu you won't see ONIE as you did before, so you can't restore it easily. However, it's possible.

## Prerequisites

* Console cable
* Downloaded OS9/OS10 image. **Make sure that the OS9 image is ONIE-based**
* USB flash drive

### Step 1. Boot to ONIE from GRUB rescue

Input following commands in grub rescue:

```
insmod ext2
insmod part_gpt
set root='(hd0,gpt2)'
set prefix=(hd0,gpt2)/grub
insmod normal
normal
```

Once entered, press **Ctrl+X** and boot into the GRUB menu with ONIE options enabled.

### Step 2. Uninstall OS

At the GRUB rescue menu press "Uninstall OS" in order to uninstall the currently install OS

### Step 3. Re-enter the ONIE shell to install OS9/OS10

After the uninstall finishes reboot and follow **Step 1** or if possible, press **Enter** a few times and enable the shell. Plug in the USB flash drive and type **fdisk -l** in order to identify your USB flash driver. After doing so, format the drive to vFAT:

```
$ mkfs.vfat /dev/<device_name>
```
After doing so, plug the flash drive into your laptop, copy the files into the flash drive, plug the flash drive into the switch and mount the drive:

```
$ mkdir /mnt/usb
$ mount -t vfat /dev/<device_name> /mnt/usb/

# Verify the contents of the folder

$ cd /mnt/usb/
$ ls
```

If you see your files there follow up by executing the following command:

```
$ onie-nos-install <image_name.bin>
```

Once this succeeds, you can reboot and use your switch. Note that the configs will be **wiped**.
