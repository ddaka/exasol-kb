---
tool_name: cos
doc_type: reference
category: system
title: "INTERNAL - Calculate Hugepages (Conversion)"
summary: "Hugepages can be set via EXAoperation or from the command line. Use EXAoperation if you can reboot the cluster, if this is not possible use the command line. In both scenarios,..."
---
# INTERNAL - Calculate Hugepages (Conversion)

Hugepages can be set via EXAoperation or from the command line. Use EXAoperation if you can reboot the cluster, if this is not possible use the command line. In both scenarios, the database must be shut down beforehand.

### Get current hugepage settings (kB)

```
[root@n0011 process]# cat /proc/sys/vm/nr_hugepages
99840
```

```
[root@n0011 process]# grep Huge /proc/meminfo
AnonHugePages:    808960 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

### Convert hugepages value from kB to GiB. Example 195GiB

```
[root@n0011 process]# bc -l
(99840*2048)/1024/1024
195.00 <- GiB
```

### Convert hugepages from GiB to kB. Example 132GiB

```
[root@n0011 process]# bc -l
(132*1024*1024)/2048
67584.00
```

### Disable transparent hugepages

```
[root@n0011 process]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

### Set hugepages without EXAoperation. Example 132GiB

```
[root@n0011 process]# echo "67584" > /proc/sys/vm/nr_hugepages
```

### Make changes persistent (use /etc/rc.local_cos inside COS or /etc/rc.local outside COS)

```
[root@n0011 process]# echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >>/etc/rc.local_cos
[root@n0011 process]# echo 'echo "67584" > /proc/sys/vm/nr_hugepages' >>/etc/rc.local_cos
```

Set EXAoperation hugepages to the same value as above otherwise hugepages will be overwritten with the old value on the next reboot
