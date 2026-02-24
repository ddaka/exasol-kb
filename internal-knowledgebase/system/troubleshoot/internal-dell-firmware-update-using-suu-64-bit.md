---
tool_name: cos
doc_type: troubleshoot
category: system
title: "INTERNAL - Dell Firmware Update using SUU 64-Bit"
summary: "This article guides you to update the firmware if a Dell Cluster."
---
# INTERNAL - Dell Firmware Update using SUU 64-Bit

## Overview

This article guides you to update the firmware if a Dell Cluster.

## Prerequisites

You need the DELL Repository Manager in order to create the Update ISO.

## Execution

## .) Transfer SUU.iso to nodes

```
  for i in {11..21}; do scp -P20 /root/SUU.iso n$i:; done
```

## .) Stop COS services

## .) Start Tmux on license node

## .) Create update script

```
 cat <<EOF>update_firmware.sh mount -o loop /root/SUU.iso /mnt export PATH=/bin:/sbin:/usr/bin/:/usr/sbin/  cd /mnt ./suu -u EOF
```

## .) Transfer script to nodes

```
  for i in {11..21}; do scp -P20 /root/update_firmware.sh n$i:; done
```

## .) Run update_firmware.sh in tmux

```
 ssh <node> -p20 bash update_firmware.sh
```
## .) Reboot node if necessary and repeat step 6 until the latest firmware is applied

check /var/log/dell/suu/update.log

## Additional Notes

 If this message appears in the logs:

```
Inventory Failure: USB is not enabled. Please enable USB and try update again
```

Workaround:

```
modprobe usb_storage
```

Sometimes /var/log/dell/suu/suu.lck won't be deleted and SUU cannot be started.

```
cd /var/log/dell/suu/
chattr -i suu.lck
rm -f suu.lck
```
