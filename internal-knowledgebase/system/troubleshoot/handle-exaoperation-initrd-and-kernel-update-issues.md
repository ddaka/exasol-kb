---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Handle EXAOperation initrd and kernel update issues"
summary: "In some cases, initrd file can get corrupted on boot. It can be appears as a \"kernel panic error\", for instance:"
---
# Handle EXAOperation initrd and kernel update issues

## Corrupted initrd File on AWS Instances

In some cases, initrd file can get corrupted on boot. It can be appears as a "kernel panic error", for instance:
```
INFO: task swapper/0:1 blocked for more than 120 seconds.
"echo 0 > /proc/sys/kernel/hung_task_timeout_secs" disables this message.
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

The corrupted initrd files should be restored from the working instances.
In a case where all the data nodes have the issue, the initrd file can be restored from the management node as follows:

1. The volume which has the faulty initrd file, should be detached from its instance.
2. For re-attaching the corresponding volume, there should be a working instance.
A new instance should be created in the same subnet of the corresponding cluster.
The detached volume should be re-attached to the newly created instance as the /dev/sda1 partition.
3. After attaching the volume, the partition can be see in the output of "lsblk" command on the console.
The name of it will be start with "nvme" and it can be found with checking its disk size.
4. The partition should be mounted to a certain path inside of the new instance.
For instance, if the name of the partition is "nvme1n1p1" and if it will be mounted to the "tmp" directory, the following command should be run:
```
mount /dev/nvme1n1p1 tmp/
```
After that, the partition should be seen in the "tmp" directory.

5.  Configuring IP address of mgmt node:
If the parameter "LICENSE_SERVER_IP" will be used in the wget and sed commands below, the management node IP should be assigned to the "LICENSE_SERVER_IP" parameter
template command on the console to do this step(if the management node's IP is 192.168.0.10):
```
export LICENSE_SERVER_IP="192.168.0.10"
```

6. Fetching new initrd.gz from mgmt.node:
```
wget --no-check-certificate -O- "https://$LICENSE_SERVER_IP:44443/initrd" > initrd.gz
```
7. Creating checksum of initrd:
```
md5sum initrd.gz | awk '{print $1}' > initrd.gz.orig.md5
```

8. Creating new directory for new initrdbuild:

```
mkdir initrdbuild
cd initrdbuild
```

9. Unzipping and extract files from initrd and cleanup:
```
gzip -d ../initrd.gz
cpio -i < initrd
rm initrd
```

10. Doing customizations for the initrd.gz needed for the cloud:

```
curl --insecure --silent "https://$LICENSE_SERVER_IP:44443/ssh-key-pub" >> etc/ssh/authorized_keys
sed -i "s/^SERVER=.*/SERVER=$LICENSE_SERVER_IP/" bin/cos-ping
sed -i 's/^from=[^ ]* *//' etc/ssh/authorized_keys

sed -i 's/exec 2>&1/&\n\**bleep** tuntap add eth1 mode tap/' exasol/init-exasol-network
sed -i 's/ sys-subsystem-net-devices-eth1.device//' etc/systemd/system/exasol-network.service
```

11. For adjustments of the exawolke-update file, the following lines should be run on the console:

```
cat <<'EOF' > exasol/exawolke-update
#!/bin/bash

if [ -n "$(grep aws_root=UUID= /proc/cmdline)" ]; then
mkdir /aws || echo "Could not create /aws directory"
UUID="$(sed "s/.*aws_root=UUID=\([a-zA-Z0-9-]*\).*/\1/g" /proc/cmdline)"
mount -U $UUID /aws || echo "Could not mount $UUID to /aws"
UUID_BOOT=$(grep /boot /aws/etc/fstab | awk '{print $1}' | sed 's/.....//')
mount -U $UUID_BOOT /aws/boot || echo "Could not mount $UUID_BOOT to /aws/boot"
mount --bind /dev /aws/dev || echo "Could not mount /dev to /aws/dev"
mount --bind /sys /aws/sys || echo "Could not mount /sys to /aws/sys"
mount --bind /proc /aws/proc || echo "Could not mount /proc to /aws/proc"
if [ -e /aws/etc/init.d/exawolke ]; then
chroot /aws /etc/init.d/exawolke update-initrd; exitcode=$?
if [ "$exitcode" = 2 ]; then
echo "reboot needed (exawolke)"
reboot
elif [ "$exitcode" = 1 ]; then
echo "Failed to update initrd (exawolke)"
fi
fi
umount /aws/dev || echo "Could not umount /aws/dev"
umount /aws/sys || echo "Could not umount /aws/sys"
umount /aws/proc || echo "Could not umount /aws/proc"
umount /aws/boot || echo "Could not umount /aws/boot"
umount /aws || echo "Could not umount /aws"
rmdir /aws || echo "Could not remove /aws"
fi
EOF
chmod +x exasol/exawolke-update
```

12. For adjustments of the exawolke-update service, the following lines should be run on the console:

```
cat <<'EOF' > ./etc/systemd/system/exawolke-update.service
[Unit]
Description=ExaWolke update service
Before=exasol.service
After=exasol-network.service

[Service]
Type=oneshot
ExecStart=/exasol/exawolke-update
KillMode=process

[Install]
WantedBy=default.target
EOF

ln -s ../exawolke-update.service ./etc/systemd/system/default.target.wants/
rm -rf lib/firmware/
```

13. Rebuilding the initrd:

```
find | cpio -H newc -o | pigz -9 > ../initrd_new.gz
```

14. Newly created initrd_new.gz and initrd.gz.orig.md5 files should be moved to the boot disk and should be replaced with the faulty ones.

15. The partition can be unmount:
```
umount tmp
```
After running the command, tmp directory won't be seen in the output of "lsblk" command.

16. The re-attached volume should be de-attached once again from the newly created instance.

17. The volume should be attached back to its original instance and the instance should be started after attachment is done. At this point, the former faulty instance should be started successfully.

18. All the corrupted volumes must be attached to the new instance one by one and copying the newly created initrd_new.gz and initrd.gz.orig.md5 files inside the corresponding partitions must be done for each volume separately, one by one.
