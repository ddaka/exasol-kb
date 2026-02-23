---
tool_name: confd_client
doc_type: guide
category: licensing
subcommands:
  - license_info
  - license_run_check
  - db_info
technical_entities:
  - license
  - license limits
  - c4 connect
  - COS
  - confd_client
  - expiration date
  - max_db_mem_size_in_gb
  - max_db_raw_data_size_in_gb
  - max_nodes_per_cluster
  - max_num_clusters
summary: >
  How to check license limits and validity — connect to COS via c4, view
  license details with license_info (contract fields and limit fields), and
  verify limits with license_run_check.
---

# Check License

This article explains how to check the limits and validity of an uploaded license.

This procedure is carried out using ConfD and c4.

---

## Connect to COS

1. Get the play ID of the deployment using `c4 ps`. For example:

```bash
c4 ps
```

```
 N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
 1  c3275f84  11    awscf   r5d.large    8.34.0      203.0.113.11    10.0.0.11    d      running  03:50:12  +∞
 1  c3275f84  12    awscf   r5d.large    8.34.0      203.0.113.12    10.0.0.12    d      running  03:50:13  +∞
 1  c3275f84  13    awscf   r5d.large    8.34.0      203.0.113.13    10.0.0.13    d      running  03:50:13  +∞
 1  c3275f84  14    awscf   r5d.large    8.34.0      203.0.113.14    10.0.0.14    d      running  03:50:13  +∞
```

2. Connect to the cluster operating system (COS) using `c4 connect -i PLAY_ID -s cos`.

```bash
./c4 connect -i c3275f84 -s cos
```

---

## View License Details

To view details about the currently installed license, use the ConfD job `license_info`.

```bash
confd_client license_info
```

```
Contract:
  comment: Test license with an expiration date.
  company_name: My Company
  distributor: Exasol
  distributor_id: 1
  expiration_date: '2026-11-05'
  license_id: 1234567890
  schema_version: 1
Limits:
  max_db_mem_size_in_gb: 1500
  max_db_raw_data_size_in_gb: 2000
  max_nodes_per_cluster: 48
  max_num_clusters: 3
```

### Contract Fields

- **`comment`** — An optional comment describing the license. This field can be empty.
- **`company_name`** — The licensee company name.
- **`distributor`** — The distributor of the license. This is currently always "Exasol".
- **`distributor_id`** — This is currently always "1".
- **`expiration_date`** — Expiration date for the license in the format `YYYY-MM-DD`. "Unlimited" or "inf" means the license does not expire.
- **`license_id`** — A unique ID number for the license.

### Limit Fields

- **`max_db_mem_size_in_gb`** — Used for licenses based on database RAM. This value limits the maximum configurable total database RAM across all databases in the cluster. The limit value is checked against the configured DB RAM at database startup. This parameter should not be confused with the Database mem size (GiB) limit in an Exasol 7.1 license, which refers to the limit of compressed data. To check the configured DB RAM, use the ConfD job `db_info`.

```bash
confd_client db_info db_name: MY_DATABASE
```

- **`max_db_raw_data_size_in_gb`** — The maximum size of raw (uncompressed) data in GiB that you can store across all databases in the cluster. Audit data is not counted towards the raw data license limit. Clusters are counted per database — n databases will count as n clusters.

- **`max_num_clusters`** — The maximum number of running clusters allowed by the license.

- **`max_nodes_per_cluster`** — The maximum number of active nodes on a running cluster. Access node and standby nodes are not counted towards this limit.

---

## Check License Limits

To verify that the databases currently running in the cluster do not exceed the limits in the license, use the ConfD job `license_run_check`. If the command returns `OK`, the limits of the license are not exceeded.

```bash
confd_client license_run_check
```

```
OK
```
