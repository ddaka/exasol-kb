---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to add storage disks as a new storage device to a running cluster with a online database"
summary: "Needs to be done per node + adopt the node UUID!!!:"
---
# How to add storage disks as a new storage device to a running cluster with a online database

Needs to be done per node + adopt the node UUID!!!:

Most of the command can be copied from the hddinit_gpt.sh from each of the nodes. If you are not 100% familiar with the stuff done here - don’t even try to do it. You will destroy the cluster.

Check /dev/exad devices before starting.

Extracted commands from (/usr/opt/EXASuite-6/EXAClusterOS-6.2.5/var/exaoperation/cluster1/nodes/n0011/hddinit_gpt.sh)

Example node n11:

* Attached as /dev/sdf nmve4
* Set temporary “install flag”
* Add d06_storage + /dev/nvme4n1
* Set “active flag”

```bash
dd if=/dev/zero of=/dev/nvme4n1 count=1024 bs=1024 || error 'Disk /dev/nvme4n1 initialization failed.'

psh bash expect -c "spawn parted /dev/nvme4n1 mklabel gpt" -c "expect \"Yes/No\"" -c "send -- \"yes\\r\"" -c "expect \"yes\\r\\nNew disk label type?\"" -c "send -- \"gpt\\r\"" -c "expect eof" || error 'Disk /dev/nvme4n1 partitioning failed.'

parted /dev/nvme4n1 mkpart primary 64s 16384s; parted /dev/nvme4n1 mkpart primary 16456s 100%

hddident -m /dev/nvme4n1p1 -u /etc/cos/node_uuid -I -N "/dev/nvme4n1" -F YES

ln -sf "/dev/nvme4n1" "/dev/nvme4n1_35C35C037D1458CED0241FA32735BEAFF862835F"

ln -sf "/dev/nvme4n1p1" "/dev/nvme4n1p1_35C35C037D1458CED0241FA32735BEAFF862835F"

ln -sf "$(readlink /dev/nvme4n1_35C35C037D1458CED0241FA32735BEAFF862835F)p2" "/dev/nvme4n1p2_35C35C037D1458CED0241FA32735BEAFF862835F"

rm -f /dev/meta6_0; ln -sf /dev/nvme4n1p1_35C35C037D1458CED0241FA32735BEAFF862835F /dev/meta6_0

rm -f /dev/meta_d06_storage_0; ln -sf /dev/nvme4n1p1_35C35C037D1458CED0241FA32735BEAFF862835F /dev/meta_d06_storage_0

rm -f /dev/disk6_0 ; ln -sf /dev/nvme4n1p2_35C35C037D1458CED0241FA32735BEAFF862835F /dev/disk6_0

rm -f /dev/disk_d06_storage_0 ; ln -sf /dev/nvme4n1p2_35C35C037D1458CED0241FA32735BEAFF862835F /dev/disk_d06_storage_0
```
