---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How-to use csbench"
summary: "> > > What is csbench? Can't we use other standard linux benchmarking tools to test disk performance? > > -> Storage benchmark tool for Exasol. No, EXAStorage has no filesystem..."
---
# How-to use csbench

### FAQ

>
> > What is csbench? Can't we use other standard linux benchmarking tools to test disk performance?
> > -> Storage benchmark tool for Exasol. No, EXAStorage has no filesystem and it is a distributed system so normal tools cannot simulate EXAStorage IO.
> > Is there a minimum throughput a single drive should have when writing small blocks ~1MiB? -> No, it is not possible to number or define a good throughput because it depends on the customer scenario. The only way to do it is to compare similar workflows, systems and setups.
> >
> >
>
>

### Prerequisites

Shell access, two DATA or ARCHIVE volumes (1x Redundancy 1, 1x Redundancy 2), tmux, csbench, vim

### CLI example create volumes (ensure there is enough free disk space available!)

```sql
# Single Node performance
# Redundancy 1, Size 60GiB, Storage disks from d04_storage, Master nodes 1, Master node n0011
csvol -c -s 15728640 -b 1 -S 64 -r 1 -h d04_storage -p rwxrw-rw- -m 1 -n 11 -H -C -B vertical

# Two Node performance in redundancy 1
# Redundancy 1, Size 60GiB (32GiB per segment per node), Storage disks from d04_storage, Master nodes 2, Master nodes n0011, n0012
csvol -c -s 15728640 -b 1 -S 64 -r 1 -h d04_storage -p rwxrw-rw- -m 2 -n 11,12 -H -C -B vertical

# Two Node performance in redundancy 2
# Redundancy 2, Size 60GiB (32GiB per segment per node), Storage disks from d04_storage, Master nodes 2, Master nodes n0011, n0012
csvol -c -s 15728640 -b 1 -S 64 -r 2 -h d04_storage -p rwxrw-rw- -m 2 -n 11,12 -H -C -B vertical
```

### Different number of disks

Tests can be done with any number of disks but for best performance it is recommended to use more than one drive. EXAStorage will start one thread per drive, more drives -> more threads -> more throughput (in theory). In my previous tests I saw a performance cap with 8 drives so using more than 8 drives won't improve the overall throughput. A good starting point would be: 1 disk per node, 2 disks per node, 4 disks per node, 8 disks per node

### Volume Redundancy 1 VS. 2

Redundancy 1 will test local R/W while redundancy 2 also tests the network and the redundancy copy R/W performance. It is recommended to run three tests:

1. Single Node performance in redundancy 1 (volume with one Master node)

2. Two Node performance in redundancy 1 (volume with two Master nodes)

3. Two Node performance in redundancy 2 (volume with two Master nodes and two redundancy segments)

### Different Block Sizes

Tests will simulate different block sizes. 1MiB will simulate "small blocks", 10MiB will simulate "medium blocks" and 100MiB will simulate "big blocks". Different block sizes create a different number of IO operations. For example, 4096MiB with a small block size of 1MiB will need ~4000 operations while the same amount of data with 100MiB block size will need ~40 operations.

### What is cstop and when should I use it?

cstop is an interactive cli tool to display various information about EXAStorage. Most common use cases are: 'p' for performance or 'r' recovery. In order to display the performance stats of all nodes press 'p' then twice 'a' (this selects all nodes) and hit enter. You can then use '.' or ',' to toggle forward or backward.

The screenshot below shows an example of node n0011 which has 4 EXAStorage drives. Let's have a closer look into this.

![](images/exa-Marcel_0-1614249912112.png)

### Iotop or iostat?

It is up to you which tool you want to use. My personal preference is iotop as it can display only active processes and it will show EXAStorage threads. The below example shows n0011 with four EXAStorage drives using 'iotop -o'

![](images/exa-Marcel_1-1614250107232.png)

The next example shows n0011 and 'iostat -m 1'. It is much harder to identify storage drives isn't it? The four storage drives are dm-3, dm-4, dm-5 and dm-6.

![](images/exa-Marcel_2-1614250211007.png)

### Execution

**CAUTION:** Please do not run csbench on any volume other than the volumes which have been created for the benchmark. csbench overwrites all data!

### Step 1. Free disk space per node

4 drives per node, examples show n0011 and n0012

Node n0011 has 18GiB of disk space per drive available (TOTAL: 4*18GiB). Check the available disk space on all nodes that you want to benchmark!

```bash
cluster50 [root@n0011 ~]# csinfo -D -i 11|grep Free
    Free sectors      : 4664320 (17.79 GiB)
    Free sectors      : 4664320 (17.79 GiB)
    Free sectors      : 4664320 (17.79 GiB)
    Free sectors      : 4664320 (17.79 GiB)
```

For this test we also want to use n0012 - so let's do the same for n0012. Node n0012 has 32GiB of disk space per drive available (TOTAL: 4*32GiB)

```bash
cluster50 [root@n0011 ~]# csinfo -D -i 12|grep Free
    Free sectors      : 8596480 (32.79 GiB)
    Free sectors      : 8596480 (32.79 GiB)
    Free sectors      : 8596480 (32.79 GiB)
    Free sectors      : 8596480 (32.79 GiB)
```

### Step 2. Create benchmark volumes

### Ssh + tmux or screen or any other terminal multiplexer

This is to ensure the session remains even if your ssh session crashes

```bash
ssh n11
tmux
```

### Single node setup in redundancy 1

```bash
csvol -c -s 15728640 -b 1 -S 64 -r 1 -h d03_storage -p rwxrw-rw- -m 1 -n 11 -H -C -B vertical
Successfully created a new volume with ID: 21
```

### Two node setup in redundancy 1

```bash
csvol -c -s 15728640 -b 1 -S 64 -r 1 -h d03_storage -p rwxrw-rw- -m 2 -n 11,12 -H -C -B vertical
Successfully created a new volume with ID: 22
```

### Two node setup in redundancy 2

```bash
csvol -c -s 15728640 -b 1 -S 64 -r 2 -h d03_storage -p rwxrw-rw- -m 2 -n 11,12 -H -C -B vertical
Successfully created a new volume with ID: 23
```

### Step 3. Run benchmark

Run small, medium and large block tests for each volume. '-p' runs each test 10 times this ensure good AVG values. Split tmux to run cstop or iotop -o in parallel (optional) -  this gives a nice live view on what is going on on IO level. Ensure csbench's '-s' is bigger than the available Raid-Controller Cache otherwise results do not demonstrate real-world workload. **CAUTION:** This can put heavy load on the IO subsystem and might crash systems, databases and other processes.

### Tmux attach or screen or any other terminal multiplexer

This is to ensure the session remains even if your ssh session crashes

```bash
ssh n11
tmux attach
```

### Example Single node benchmark (4096MiB of data)

```bash
ssh n11 csbench -v 21 -s 4294967200 -m 1000000 -M 1000000 -b rwspSP --seed=1 -p 10
ssh n11 csbench -v 21 -s 4294967200 -m 10000000 -M 10000000 -b rwspSP --seed=1 -p 10
ssh n11 csbench -v 21 -s 4294967200 -m 100000000 -M 100000000 -b rwspSP --seed=1 -p 10
```

### Example two node benchmark in redundancy 1 (4096MiB of data)

 Ensure to run csbench on all nodes included in this volume - remember we want to simulate parallel database workload. You can either use cosexec as shown below or separate ssh sessions to the different nodes - most important: run csbench at the same time!

```bash
cosexec -rtn 11,12 csbench -v 22 -s 4294967200 -m 1000000 -M 1000000 -b rwspSP --seed=1 -p 10
cosexec -rtn 11,12 csbench -v 22 -s 4294967200 -m 10000000 -M 10000000 -b rwspSP --seed=1 -p 10
cosexec -rtn 11,12 csbench -v 22 -s 4294967200 -m 100000000 -M 100000000 -b rwspSP --seed=1 -p 10
```

### Example two node benchmark in redundancy 2 (4096MiB of data)

```bash
cosexec -rtn 11,12 csbench -v 23 -s 4294967200 -m 1000000 -M 1000000 -b rwspSP --seed=1 -p 10
cosexec -rtn 11,12 csbench -v 23 -s 4294967200 -m 10000000 -M 10000000 -b rwspSP --seed=1 -p 10
cosexec -rtn 11,12 csbench -v 23 -s 4294967200 -m 100000000 -M 100000000 -b rwspSP --seed=1 -p 10
```

### Step 4. The results

So what do the below numbers mean? This example shows the test results from n0011 and data volume v0021 (1 master node segment on n0011) in redundancy 1.

1st: 4096MiBs of data have been divided by ~100MiBs blocks which results on ~42 IO operations.

2nd: node n0011 has four disks: to get the throughput of a single disk divide below results for WRITE and READ by 4!

3rd: volume redundancy 2 will decrease the throughput (may be even half below numbers!)

4th: look for anomalies e.g. in the below example READ SEQ is much faster than all other operations. Does it make sense or is it some caching effects?

csbench parameters used:

-v = volume id

-s = size in bytes

-m = min-size

-M = max-size

-b = bench-mode: read, write, sequentiell, parallel

--seed = use random numbers (entropy: srand)

-p = number of passes

```bash
Using local offset 0 (0.00 B).
created new cache of size 4294967200 and alignment 4096.
====== pass 1 ======
Created 42 blocks with a total size of 3.91 GiB.
--- Doing sequ. write (3.91 GiB in 42 blocks) ---
--- Doing sequ. read (3.91 GiB in 42 blocks) ---
--- Doing sequ. parallel write (3.91 GiB in 42 blocks) ---
--- Doing sequ. parallel read (3.91 GiB in 42 blocks) ---
--- Doing random write (3.91 GiB in 42 blocks) ---
--- Doing random read (3.91 GiB in 42 blocks) ---
--- Doing random parallel write (3.91 GiB in 42 blocks) ---
--- Doing random parallel read (3.91 GiB in 42 blocks) ---
------------------------
- Volume ID       : 21
- Alignment       : 4096
- Passes          : 1
- Seed            : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (3.91 GiB in 42 blocks) :  145 |  417 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (3.91 GiB in 42 blocks) :  110 |  143 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (3.91 GiB in 42 blocks) : 1263 |  404 |    0 MiB/s  (seq|par|gathered)
- READ  RND (3.91 GiB in 42 blocks) :  283 |  323 |    0 MiB/s  (seq|par|gathered)
------------------------
```

### Is the performance now good or bad?

Again, it depends on the context, e.g. is the system on-prem or virtualised, is it SSDs or SAS or regular spinning drives and so on. Compare the results to a system where you know it has good performance and equal specs - I know this might not be an satisfying answer but let me give you some more examples from a recent test for a customer that migrated from on-prem to GCP.

### Real world examples Storage

### Disk performance on-prem VS. GCP

Write speed of the disks is factor ~1.5-2.8 slower compared to on-prem.

Setup: 50GiB Data volume in R1 on each reserve node. 1MiB, 10MiB, 100MiB. 10 rounds each.

```bash
csbench -v 3 -s 4294967200 -m 1000000 -M 1000000 -b rwspSP --seed=1 -p 10
csbench -v 3 -s 4294967200 -m 10000000 -M 10000000 -b rwspSP --seed=1 -p 10
csbench -v 3 -s 4294967200 -m 100000000 -M 100000000 -b rwspSP --seed=1 -p 10
```

### On-prem 7x1200GiB SAS 15K - GOOD

```bash
------------------------
- Volume ID : 29
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 976.56 KiB / 976.56 KiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.99 GiB in 42790 blocks) : 821 | 1749 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.99 GiB in 42790 blocks) : 827 | 935 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.99 GiB in 42790 blocks) : 609 | 1581 | 0 MiB/s (seq|par|gathered)
- READ RND (39.99 GiB in 42790 blocks) : 119 | 330 | 0 MiB/s (seq|par|gathered)
------------------------

------------------------
- Volume ID : 29
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 9.54 MiB / 9.54 MiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.96 GiB in 4290 blocks) : 1851 | 1793 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.96 GiB in 4290 blocks) : 1540 | 1594 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.96 GiB in 4290 blocks) : 1320 | 1900 | 0 MiB/s (seq|par|gathered)
- READ RND (39.96 GiB in 4290 blocks) : 685 | 1146 | 0 MiB/s (seq|par|gathered)
------------------------

------------------------
- Volume ID : 29
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.12 GiB in 420 blocks) : 1388 | 1638 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.12 GiB in 420 blocks) : 1522 | 1739 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.12 GiB in 420 blocks) : 1774 | 2383 | 0 MiB/s (seq|par|gathered)
- READ RND (39.12 GiB in 420 blocks) : 1753 | 2204 | 0 MiB/s (seq|par|gathered)
------------------------
```

### On-prem 3x3.3TiB Read-intense SATA SSDs - GOOD

```bash
------------------------
- Volume ID       : 2
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 976.56 KiB / 976.56 KiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.99 GiB in 42790 blocks) :  887 | 1681 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.99 GiB in 42790 blocks) :  869 | 1683 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.99 GiB in 42790 blocks) : 1104 | 2003 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.99 GiB in 42790 blocks) : 1085 | 1962 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 2
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 9.54 MiB / 9.54 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.96 GiB in 4290 blocks) : 2766 | 6284 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.96 GiB in 4290 blocks) : 2737 | 6332 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.96 GiB in 4290 blocks) : 3208 | 5075 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.96 GiB in 4290 blocks) : 3202 | 5069 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 2
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.12 GiB in 420 blocks) : 4510 | 6508 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.12 GiB in 420 blocks) : 4546 | 6484 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.12 GiB in 420 blocks) : 4601 | 6024 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.12 GiB in 420 blocks) : 4620 | 5961 |    0 MiB/s  (seq|par|gathered)
------------------------
```

### GCP 9x1024GiB SSD - BAD

```bash
------------------------
- Volume ID : 3
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 976.56 KiB / 976.56 KiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.99 GiB in 42790 blocks) : 291 | 708 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.99 GiB in 42790 blocks) : 290 | 706 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.99 GiB in 42790 blocks) : 584 | 1135 | 0 MiB/s (seq|par|gathered)
- READ RND (39.99 GiB in 42790 blocks) : 591 | 1128 | 0 MiB/s (seq|par|gathered)
------------------------

------------------------
- Volume ID : 3
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 9.54 MiB / 9.54 MiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.96 GiB in 4290 blocks) : 801 | 961 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.96 GiB in 4290 blocks) : 817 | 961 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.96 GiB in 4290 blocks) : 1204 | 1192 | 0 MiB/s (seq|par|gathered)
- READ RND (39.96 GiB in 4290 blocks) : 1204 | 1195 | 0 MiB/s (seq|par|gathered)
------------------------

------------------------
- Volume ID : 3
- Alignment : 4096
- Passes : 10
- Seed : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.12 GiB in 420 blocks) : 1040 | 1114 | 0 MiB/s (seq|par|scattered)
- WRITE RND (39.12 GiB in 420 blocks) : 1050 | 1111 | 0 MiB/s (seq|par|scattered)
- READ SEQ (39.12 GiB in 420 blocks) : 1203 | 1203 | 0 MiB/s (seq|par|gathered)
- READ RND (39.12 GiB in 420 blocks) : 1203 | 1203 | 0 MiB/s (seq|par|gathered)
------------------------
```

### GCP 5x2048GIB SSD - BAD

```bash
------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 976.56 KiB / 976.56 KiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.99 GiB in 42790 blocks) :  372 |  895 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.99 GiB in 42790 blocks) :  367 |  893 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.99 GiB in 42790 blocks) :  574 | 1143 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.99 GiB in 42790 blocks) :  578 | 1143 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 9.54 MiB / 9.54 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.96 GiB in 4290 blocks) :  868 | 1127 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.96 GiB in 4290 blocks) :  862 | 1136 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.96 GiB in 4290 blocks) : 1204 | 1199 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.96 GiB in 4290 blocks) : 1204 | 1199 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.12 GiB in 420 blocks) : 1063 | 1180 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.12 GiB in 420 blocks) : 1068 | 1189 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.12 GiB in 420 blocks) : 1203 | 1203 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.12 GiB in 420 blocks) : 1203 | 1203 |    0 MiB/s  (seq|par|gathered)
------------------------
```

### GCP 5x2048GiB HDD - BAD

```bash
------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 976.56 KiB / 976.56 KiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.99 GiB in 42790 blocks) :  372 | 1009 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.99 GiB in 42790 blocks) :  366 | 1009 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.99 GiB in 42790 blocks) :  578 | 1103 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.99 GiB in 42790 blocks) :  556 | 1010 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 9.54 MiB / 9.54 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.96 GiB in 4290 blocks) :  884 | 1131 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.96 GiB in 4290 blocks) :  889 | 1138 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.96 GiB in 4290 blocks) : 1172 | 1159 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.96 GiB in 4290 blocks) : 1172 | 1159 |    0 MiB/s  (seq|par|gathered)
------------------------

------------------------
- Volume ID       : 1
- Alignment       : 4096
- Passes          : 10
- Seed            : 1
- Min./max. block : 95.37 MiB / 95.37 MiB
- Random offset   : 0.00 B
- I/O window size : 4.00 GiB
- WRITE SEQ (39.12 GiB in 420 blocks) : 1065 | 1170 |    0 MiB/s  (seq|par|scattered)
- WRITE RND (39.12 GiB in 420 blocks) : 1075 | 1176 |    0 MiB/s  (seq|par|scattered)
- READ  SEQ (39.12 GiB in 420 blocks) : 1170 | 1170 |    0 MiB/s  (seq|par|gathered)
- READ  RND (39.12 GiB in 420 blocks) : 1170 | 1170 |    0 MiB/s  (seq|par|gathered)
------------------------
```
