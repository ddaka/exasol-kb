---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Step‑by‑Step Guide: Fixing **Cache‑Disk Size Mismatch** After DB Scale‑Rollback Failure"
summary: "> **Scenario** > A scale‑rollback failed, the database won’t start, and `/exa/logs/cored/exainit.log` on node `n11` shows a `Failed to add device /dev/ephemeral1` error. > Goal:..."
---
# Step‑by‑Step Guide: Fixing **Cache‑Disk Size Mismatch** After DB Scale‑Rollback Failure

> **Scenario**
> A scale‑rollback failed, the database won’t start, and `/exa/logs/cored/exainit.log` on node `n11` shows a `Failed to add device /dev/ephemeral1` error.
> Goal: verify the issue, correct the cache‑disk size in **EXAConf**, commit the change, and restart the DB.
>
> **Important:** Before starting the recovery, always check the **desired size** reported in the SaaS control plane.
>
> - Focus on getting the system to **start with whatever size it currently has**.
> - After the database is online and healthy, resize it to match the **desired size** shown in the SaaS dashboard.
> - **Do not** run `infra_db_scale` on a broken system—this path is untested and can leave the environment in an unknown state.
>   Once SaaS reports the correct size, follow the steps below to verify the disk sizes and bring the DB back online.

---

## Connect to the cluster

```bash
# From your workstation
aws ssm start-session --target <instance‑id‑of‑n10-or-n1X>
```

> _(Use the IAM role/profile documented in the AWS‑SSM Access Guide.)_

---

## Verify that the scale‑rollback failed

```bash
exajobs ls --name infra_db_scale -M 500

# Example output for infra_db_scale jobs:
# ---------------------------------------------------------------------------------------------
# job_id       job_name                 start_date job_intent job_state revision job_result
# 10.1440810 infra_db_scale 2025-07-21 13:09:29 +00:00    mod_par  archived      N/A         OK
# 10.1438256 infra_db_scale 2025-07-21 08:49:29 +00:00    mod_par  archived      N/A         OK
# 10.1398040 infra_db_scale 2025-07-18 13:23:26 +00:00    mod_par  archived      N/A         OK
# 10.822696 infra_db_scale 2025-06-27 09:06:26 +00:00    mod_par  archived      N/A         OK
# 10.822259 infra_db_scale 2025-06-27 08:41:14 +00:00    mod_par  archived      N/A         OK
# 10.803931 infra_db_scale 2025-06-26 16:22:59 +00:00    mod_par  archived      N/A         OK
# 10.803417 infra_db_scale 2025-06-26 15:29:07 +00:00    mod_par  archived      N/A         OK
# 10.803231 infra_db_scale 2025-06-26 15:07:44 +00:00    mod_par  archived      N/A         OK
# 10.760493 infra_db_scale 2025-06-23 15:25:27 +00:00    mod_par  archived      N/A  Exception
# ---------------------------------------------------------------------------------------------

# The job must have "Exception" as its result; if none of the jobs show "Exception," this guide does not apply.
```

## Confirm EC2 instance sizes

Check the AWS EC2 console or run:

```bash
aws ec2 describe-instances --instance-ids <ids>
```

Ensure each DB node’s instance type matches the _intended_ size.

```text
NAME    TSHIRT_SIZE INSTANCE_TYPE
------- ----------- -------------
XSmall  XS          r5d.large
Small   S           r5d.xlarge
Medium  M           r5d.2xlarge
Large   L           r5d.4xlarge
XLarge  XL          r5d.8xlarge
2XLarge 2XL         r5d.16xlarge
3XLarge 3XL         r5d.24xlarge
```

---

## Locate the cache‑disk error (execute from `n10`, will grep all nodes)

```bash
psh 'grep -A3 -B2 "Failed to add device" /exa/logs/cored/exainit.log'
```

You should see something like:

```text
[2025-07-22 07:06:23.645538 +00:00] stage4: Run service storage_ephemeral
Traceback (most recent call last):
  ...
Exception: Failed to add device /dev/ephemeral1: operation failed
```

---

## Inspect current cache‑disk values in **EXAConf**

```bash
# Still on node n11
grep Cache /exa/etc/EXAConf*
```

Example output showing size changes between different EXAConf versions (useful for identifying the appropriate cache‑disk size):

```text
/exa/etc/EXAConf:      CacheVolumeDisk = disk2:937.5 GiB
/exa/etc/EXAConf.6:    CacheVolumeDisk = disk2:468.75 GiB
/exa/etc/EXAConf.check: CacheVolumeDisk = disk2:58.5938 GiB
```

> **Sizing advice**
> SaaS node sizes follow an (approximate) doubling scheme: **XS ≈ 58 GiB, S ≈ 117 GiB, M ≈ 236 GiB, L ≈ 472 GiB, XL ≈ 937 GiB, XXL ≈ 1.9 TiB, XXXL ≈ 3.8 TiB**. The cache‑disk size must correspond to the EC2 instance class (XS → `.large`, S → `.xlarge`, etc.). If in doubt, inspect `CacheVolumeDisk` on another healthy node of the desired size and copy that value.

---

## Edit **EXAConf**

1. Open the file:

   ```bash
   sudo vim /exa/etc/EXAConf
   ```

2. In the **Global** section set:

   ```ini
   Checksum = commit
   ```

3. In the **DB** section for the affected node(s) set:

   ```ini
   CacheVolumeDisk = disk2:<expected‑size‑GiB>
   ```

   _Example for XS:_ `disk2:58.5938 GiB` — **❗ each worker has its own section and CacheVolumeDisk value; make sure you edit the correct one!**

4. Save and quit (`:wq`).

---

## Commit the configuration

```bash
exaconf commit
```

The command recalculates checksums and distributes the new disk layout.

---

## Restart / check the database

```bash
confd_client db_start db_name: <DB‑name>   # if it didn’t auto‑start
dwad_client list           # confirm it is online (see below)
```

```text
root@n10:~# dwad_client list
List known systems.
+++++++++++++++++++++++++++++++++++++++++++++
Name: <DB‑NAME>
ID: <DB‑ID>
State: running
Connection state: up
<state comments> state comments available.
```

---

## Run post‑checks

```bash
cosexec -art bash -c 'grep Cache /exa/etc/EXAConf'
```

Verify that **all** nodes show the _same_ `CacheVolumeDisk` size. If any node differs, repeat **EXAConf** editing and `exaconf commit`.

---

## (Optional) Start the database via `confd_client`

```bash
confd_client db_start db_name: <DB-NAME>
```

Use this command only if the database did not auto‑start or if `exaoperation start` is unavailable.

---

## (Optional) Trigger a new scale‑out to the desired size

```bash
confd_client infra_db_scale db_name: <DB-NAME> instance_type: <INSTANCE-TYPE>
```

Replace `<DB-NAME>` and `<INSTANCE-TYPE>` (e.g., `r5d.8xlarge`) with the customer’s target settings.

**After running the command:**

1. Open the **AWS CloudFormation** console (or run `aws cloudformation describe-stack-events --stack-name <STACK-NAME>`) and monitor the stack for `UPDATE_IN_PROGRESS` → `UPDATE_COMPLETE` events.
   _It can take several minutes before any events appear._
2. Watch the **EC2** console: the database will stop first, the nodes will transition to `stopped`, then new update events appear in the CloudFormation stack, and finally the **instance type** for each node changes to the requested size.
3. After the CloudFormation update completes, the DB will start automatically. Verify the resize job succeeded with `exajobs ls` (as in step 2) and confirm the DB is running with `dwad_client list`.

---

## Document the incident

Add to the ticket:

- Failed job ID + error excerpt.
- Original vs. corrected `CacheVolumeDisk` value.
- Timestamp of `exaconf commit` and DB restart.
- Whether the database had to be started manually (note the command used, e.g. `confd_client db_start`).
- Whether a new scale‑out was triggered (command used and target instance type).
