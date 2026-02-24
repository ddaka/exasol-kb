---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "INTERNAL - How-to enlarge EXAStorage disks on NGA (LVM-based)"
summary: "Use **dmesg** to verify that the kernel recognized the new size of the block device"
---
# INTERNAL - How-to enlarge EXAStorage disks on NGA (LVM-based)

## Stop services

### SSH into the cluster process namespace (ssh port depends on the config -> check EXAConf)

```
[root ~#] ssh -p2222 localhost
```

### Create a remote full backup

```
[root@n11 ~]# dwad_client start storage-backup exa_db1 10001 0 ""
```

### Stop the database

```
[root@n11 ~]# dwad_client stop-wait exa_db1
```

### Stop EXAStorage

```
[root@n11 ~]# csctrl -d
```

### Check Exasol service status with **systemd**

```
[root ~#]  systemctl status exasol.service
 exasol.service - exasol
   Loaded: loaded (/etc/systemd/system/exasol.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-01-13 03:01:23 EST; 3 weeks 6 days ago
 Main PID: 1477 (exact)
    Tasks: 360 (limit: 26213)
   Memory: 961.6M
   CGroup: /system.slice/exasol.service
           ├─ 1477 /opt/image/usr/opt/EXASuite-6/EXAClusterOS-6.2.4/sbin/exact mount /opt/cl1 /opt/image /opt/exa init-sc -i 11
           ├─ 1491 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/cos_cored -n 11 --renice -10 --auth-sock-dir /var/lock --oom-adjustment -350 -m 224.0.0.1 -s 10.70.10.193 -t 10001 -l /exa/logs/cored --default-file-mode 644 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/ex>
           ├─ 2529 tail -n0 -q -F /exa/logs/cored/exainit.log /exa/logs/cored/cored.log
           ├─ 2549 python2 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/super_cored -d -a
           ├─ 2896 /usr/sbin/sshd -D -o PermitRootLogin=yes -o MaxStartups=300:30:800 -h /exa/etc/hostkey -p 2222
           ├─ 2902 /sbin/rsyslogd -n -f /exa/etc/rsyslog.conf
           ├─ 2903 /sbin/crond -n -s
           ├─ 2904 /bin/sh /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/sysd
           ├─ 2948 /usr/opt/EXASuite-6/EXARuntime-6.2.4/bin/python2 -OO /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/bucketfsd --config=/exa/etc/bucketfs.cfg_bfsdefault --user=500:500
           ├─ 2950 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/logd --default-log-dir /exa/logs/logd
           ├─ 2951 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/lockd
           ├─ 2955 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/dwad --backupfile /exa/metadata/dwad/dwad.dump --store-config /exa/metadata/dwad/dwad.dump --store-interval 10
           ├─ 3016 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/cos_storage /exa/etc/cos_storage.conf
           ├─ 3051 /usr/opt/EXASuite-6/EXARuntime-6.2.4/bin/python2.7 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/confd -v
           ├─ 3088 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/controller -dbName DB1 -commMain n11 -commNPROC 1 -port 8563 -mode rw -outputdir /exa/logs/db/DB1/ -netmask= -auditing_enabled=0 -lockslb=1 -sandboxPath=/usr/opt/mountjail -cosLogErrors=0 -bucketFSConfigPath=/exa>
           ├─ 3120 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/pddserver -current_process_nr=1
           ├─ 3219 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/objectserver -current_process_nr=2
           ├─ 3289 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exasqllog -current_process_nr=4
           ├─ 3318 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/loaderd -current_process_nr=5
           ├─ 3341 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exaetl -current_process_nr=1024
           ├─ 3342 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exacs -current_process_nr=3
           ├─11504 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exasql -current_process_nr=148
           └─14384 sleep 5
```

### Stop Exasol service

```
# leave COS process namespace
[root@n11 ~]# exit
[root ~#] systemctl stop exasol.service
 exasol.service - exasol
   Loaded: loaded (/etc/systemd/system/exasol.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2021-02-09 07:33:31 EST; 8s ago
  Process: 14562 ExecStop=/opt/control/exasol stop (code=exited, status=0/SUCCESS)
  Process: 1477 ExecStart=/opt/control/exasol start (code=exited, status=9)
 Main PID: 1477 (code=exited, status=9)

Feb 09 07:33:11 oel systemd[1]: Stopping exasol...
Feb 09 07:33:30 oel exasol[14562]: Stop system DB1 and wait for shutdown.
Feb 09 07:33:31 oel exasol[14562]: Successfully stopped EXAStorage.
Feb 09 07:33:31 oel systemd[1]: exasol.service: Main process exited, code=exited, status=9/n/a
Feb 09 07:33:31 oel systemd[1]: exasol.service: Failed with result 'exit-code'.
Feb 09 07:33:31 oel systemd[1]: Stopped exasol.
```

## Resize disk(s)

### Resize the disk(s) that is (are) being used for EXAStorage (check EXAConf for details on which device/disk is used)

Use **dmesg** to verify that the kernel recognized the new size of the block device

```
[root ~#] dmesg
[2349627.820556] virtio_blk virtio1: [vda] new size: 419430400 512-byte logical blocks (215 GB/200 GiB)
[2349627.820565] vda: detected capacity change from 107374182400 to 214748364800
```

### Resize block device(s) used by LVM (repeat steps for all devices being used for EXAStorage)

### Resize partition(s)

```
[root ~]# parted /dev/vda
GNU Parted 3.2
Using /dev/vda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  1075MB  1074MB  primary  xfs          boot
 2      1075MB  107GB   106GB   primary               lvm

(parted) resizepart 2
End?  [107GB]? 200GB
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  1075MB  1074MB  primary  xfs          boot
 2      1075MB  200GB   199GB   primary               lvm
(parted) quit
[root ~]# partprobe
```

### Identify the block device (s) being used by LVM

```
[root ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/vda2
  VG Name               ol
  PV Size               <99.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              25343
  Free PE               0
  Allocated PE          25343
  PV UUID               DZ3cyy-WdXC-WOj0-lG3X-4YLn-9Bvk-Fc6Mf5
```

### Resize device(s)

```
[root ~]# pvresize /dev/vda2
  Physical volume "/dev/vda2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

### Verify that the resize worked

```
[root ~#] pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/vda2  ol lvm2 a--  185.26g <86.27g
```

### Extend the logical volume(s)

```
[root ~#] lvextend -L+30G /dev/ol/lvol0
```

## Start services

### Start the Exasol service using systemd

```
[root ~]# systemctl start exasol
 exasol.service - exasol
   Loaded: loaded (/etc/systemd/system/exasol.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2021-02-09 08:11:04 EST; 6min ago
  Process: 7765 ExecStop=/opt/control/exasol stop (code=exited, status=0/SUCCESS)
 Main PID: 7875 (exact)
    Tasks: 331 (limit: 219545)
   Memory: 485.2M
   CGroup: /system.slice/exasol.service
           ├─7875 /opt/image/usr/opt/EXASuite-6/EXAClusterOS-6.2.4/sbin/exact mount /opt/cl1 /opt/image /opt/exa init-sc -i 11
           ├─7876 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/cos_cored -n 11 --renice -10 --auth-sock-dir /var/lock --oom-adjustment -350 -m 224.0.0.1 -s 10.70.10.193 -t 10001 -l /exa/logs/cored --default-file-mode 644 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/exa>
           ├─7896 tail -n0 -q -F /exa/logs/cored/exainit.log /exa/logs/cored/cored.log
           ├─7912 python2 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/super_cored -d -a
           ├─8092 /usr/sbin/sshd -D -o PermitRootLogin=yes -o MaxStartups=300:30:800 -h /exa/etc/hostkey -p 2222
           ├─8094 /sbin/rsyslogd -n -f /exa/etc/rsyslog.conf
           ├─8095 /sbin/crond -n -s
           ├─8099 /bin/sh /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/sysd
           ├─8136 /usr/opt/EXASuite-6/EXARuntime-6.2.4/bin/python2 -OO /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/bucketfsd --config=/exa/etc/bucketfs.cfg_bfsdefault --user=500:500
           ├─8138 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/logd --default-log-dir /exa/logs/logd
           ├─8139 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/lockd
           ├─8143 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/dwad --backupfile /exa/metadata/dwad/dwad.dump --store-config /exa/metadata/dwad/dwad.dump --store-interval 10
           ├─8216 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/cos_storage /exa/etc/cos_storage.conf
           ├─8261 /usr/opt/EXASuite-6/EXARuntime-6.2.4/bin/python2.7 /usr/opt/EXASuite-6/EXAClusterOS-6.2.4/libexec/confd -v
           ├─8312 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/controller -dbName DB1 -commMain n11 -commNPROC 1 -port 8563 -mode rw -outputdir /exa/logs/db/DB1/ -netmask= -auditing_enabled=0 -lockslb=1 -sandboxPath=/usr/opt/mountjail -cosLogErrors=0 -bucketFSConfigPath=/exa/>
           ├─8361 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/pddserver -current_process_nr=1
           ├─8455 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/objectserver -current_process_nr=2
           ├─8512 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exasqllog -current_process_nr=4
           ├─8551 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/loaderd -current_process_nr=5
           ├─8572 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exaetl -current_process_nr=1024
           ├─8574 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exacs -current_process_nr=3
           ├─8613 /usr/opt/EXASuite-6/EXASolution-6.2.4/bin/exasql -current_process_nr=7
           └─9096 sleep 5
```

## Enlarge the EXAStorage device(s)

```
[root ~]# ssh -p2222 localhost
[root@n11 ~]# cshdd -E -n 11 -h /dev/ol/lvol0
```

### Verify new device size

```
[root@n11 ~]# csinfo -D

=== Node 11 (n11) ===

 ** HDD 0 **
    Name              : /dev/ol/lvol0
    Metadata          : superblock (4.00 MiB)
    Type              : disk1
    State             : ONLINE
    I/O error         : false
    CRC error         : false
    Read-only         : false
    Checksum algo     : FLETCHER2
    Sync checksums    : true
    Flags             : O_DIRECT
    Sector-size       : 4096
    Sectors           : 8814972 (33.63 GiB)
    Data sectors      : 8804352 (33.59 GiB)
    Free sectors      : 6969344 (26.59 GiB)

 >Increase info-level to see allocated partitions.
```

## Start the database

```
[root@n11 ~]# dwad_client start-wait exa_db1
```
