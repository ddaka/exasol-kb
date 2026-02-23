# Exasol Sizing Guidelines (On-Premise)

**Category:** Configuration  
**Topic:** Capacity Planning, Performance, Hardware Sizing, On-Premise  
**Keywords:** sizing, capacity planning, RAM, storage, disk space, performance, cluster, nodes, on-premise, hardware, backup  
**Source:** [Exasol On-Premise Sizing Documentation](https://docs.exasol.com/db/latest/administration/on-premise/sizing.htm)

## Overview

This document explains how to determine the disk space and RAM that will be required for your Exasol database in an **on-premise deployment**. Proper sizing is critical for optimal performance and ensuring sufficient capacity for data growth.

For **AWS cloud deployments**, see the [AWS Sizing Guidelines](exasol_sizing_guidelines.md).

## Factors That Impact Sizing

Several key factors determine the storage disk space and RAM requirements:

### 1. Expected Volume of Raw Data

The volume of uncompressed (raw) data has the **largest impact** on required storage disk space.
- Larger data volumes require more storage space
- Table data is **automatically compressed** by Exasol to optimize disk usage
- Typical compression ratio: **~2.5:1**

**Example**: 2500 GiB raw data → ~1000 GiB compressed

### 2. Performance Requirements

The amount of RAM allocated for the database (**DB RAM**) directly impacts performance.

**Rule of thumb**: Allocate **10% of uncompressed data volume** as DB RAM
- For 2500 GiB raw data → minimum 250 GiB DB RAM
- More precise calculation: Use **20% of compressed data** for better performance
- Use system tables for actual running systems

### 3. Cluster Redundancy

Redundancy level affects required storage space:
- **Redundancy 1**: Data stored once (no redundancy)
- **Redundancy 2**: Data stored in two segments (doubles storage requirements)
- Default recommendation: **Redundancy 2** for production systems

### 4. Number of Reserve Nodes

Reserve nodes are used in case of node failure:
- Must have **same hardware configuration** as active nodes
- Not included in active cluster during normal operation
- Essential for high availability
- See [Fail Safety (On-Prem)](https://docs.exasol.com/db/latest/planning/fail_safety/fail_safety_on_premise.htm)

### 5. Backup Strategy

Backup strategy impacts storage requirements:
- **Local backups** (stored in cluster): Require additional storage
- **Remote backups** (stored externally): No local storage impact
- Local backups with redundancy 2: Storage requirements double
- See [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)

### 6. Business Continuity Strategy

The architecture of your business continuity solution influences sizing:
- **Reserve cluster in separate data center**: Doubles hardware costs
- **Disaster recovery site**: Additional infrastructure requirements
- See [Business Continuity](https://docs.exasol.com/db/latest/planning/business_continuity.htm)

### 7. Operating System Reserved RAM

**Recommendation**: Reserve **10% of total RAM** for the operating system on each node.

This must be considered when calculating DB RAM allocation:
- Total node RAM = DB RAM + OS RAM
- OS RAM = 10% of total node RAM
- Example: 1000 GiB node → 900 GiB max DB RAM, 100 GiB OS RAM

---

## Database Disk Space

The total required disk space for an Exasol system is the sum of:
- **Database disk space** (data, indexes, statistics)
- **Backup disk space** (if using local backups)

### Components of Database Disk Space

#### 1. Compressed Data Volume

- Table data is **automatically compressed**
- **Typical compression ratio**: ~2.5:1
- Example: 2500 GiB raw → ~1000 GiB compressed

#### 2. Index Volume

Indexes are automatically created and maintained:
- Size depends on data model and query patterns
- Range: **2% to over 100%** of compressed data
- **Typical value**: **15% of compressed data**
- Can be checked in running systems via `EXA_DB_SIZE_*` system tables (`AUXILIARY_SIZE_*` columns)

#### 3. Statistical and Auditing Data Volume

- **Statistical data**: Small footprint
- **Auditing data**: Can be significant if enabled
  - Each login recorded
  - Each query recorded
  - Recommended: **5% of compressed data**

#### 4. Temporary Data and Fragmentation

Additional space needed for:
- **Temporary data**: Intermediate query results that don't fit in DB RAM
- **Volume fragmentation**: Persistent volume fragmentation over time

**Recommended headroom**: **60% of compressed data** (without redundancy)

### Database Disk Space Formula

```
Required Database Disk Space = 
  (Compressed Data Volume + Index Volume + Statistical/Auditing Data Volume) × Redundancy 
  + Temporary Data and Fragmentation Headroom
```

### Example Calculation

**Input Parameters:**
- Raw data volume: **2500 GiB**
- Redundancy: **2**
- Compression ratio: **2.5**
- Index volume: **15%** of compressed data
- Statistical/auditing: **5%** of compressed data
- Temp/fragmentation headroom: **60%** of compressed data (without redundancy)

**Calculation Steps:**

| Component | Formula | Result |
|-----------|---------|--------|
| Compressed data | 2500 GiB ÷ 2.5 | **1000 GiB** |
| Index volume | 1000 GiB × 15% | **150 GiB** |
| Statistical and auditing data | 1000 GiB × 5% | **50 GiB** |
| **Total data volume (net)** | 1000 + 150 + 50 | **1200 GiB** |
| **Total with redundancy** | 1200 GiB × 2 | **2400 GiB** |
| Headroom for temp data and fragmentation | 1000 GiB × 60% | **720 GiB** |
| **Required database disk space** | 2400 + 720 | **3200 GiB** |

---

## Backup Disk Space

The required backup disk space depends on your backup configuration.

### Backups Stored Outside Cluster (Remote Backups)

**Formula:**
```
Required Backup Disk Space = 
  (Full Backup Size × (Number of Full Backups + 1)) 
  + (Incremental Backup Size × Number of Incremental Backups)
```

**Note**: Creating a new backup does not remove the old backup, so headroom for an extra backup during creation is needed (Number of Full Backups + 1).

### Backups Stored in Cluster (Local Backups)

**Formula:**
```
Required Backup Disk Space = 
  ((Full Backup Size × (Number of Full Backups + 1)) 
  + (Incremental Backup Size × Number of Incremental Backups)) × 2
```

**Important**: Local backups with redundancy 2 require 2× the space.

#### Example Calculation (Local Backups)

**Input Parameters:**
- Total net data volume: **1200 GiB**
- Number of full backups: **2**
- Number of incremental backups: **3**
- Maximum incremental backup size: **100% of full backup**
- Cluster-internal backup: **Yes**
- Backup redundancy: **2**

**Calculation Steps:**

| Component | Formula | Result |
|-----------|---------|--------|
| Total data volume (net) | Compressed + Index + Stats | **1200 GiB** |
| Full backup data size | 1200 GiB × 2 backups | **2400 GiB** |
| Incremental backup data size | 1200 GiB × 3 backups | **3600 GiB** |
| Required backup space (without redundancy) | 2400 + 3600 | **6000 GiB** |
| **Required backup space (with redundancy 2)** | 6000 × 2 | **12000 GiB** |

**Note**: Because different versions of an object can be accessed by multiple queries run by different users, the backup can be larger than the physical layout of the objects themselves. We recommend including additional space to allow for this in the archive volume.

---

## Database RAM (DB RAM)

An Exasol database typically performs well with database RAM of **10% of the raw (uncompressed) data volume**. However, several other factors also affect the required DB RAM.

### Factors Affecting DB RAM Requirements

#### 1. Index Volume

Indexes are automatically created and maintained:
- Range: **2% to over 100%** of compressed data
- **Typical value**: **15% of compressed data**
- Higher index volumes negatively impact performance and require more DB RAM
- Check actual values: `AUXILIARY_SIZE_*` columns in `EXA_DB_SIZE_*` statistical tables

#### 2. Temporary Data

RAM for intermediate query results:
- When results don't fit in DB RAM → swapped to temp volume (significant performance degradation!)
- **Recommended**: Reserve extra headroom for TEMP DB RAM
- Prevents performance deterioration from disk swapping

#### 3. User-Defined Functions (UDFs)

UDFs execute in parallel and require RAM:
- **Parallel execution**: As many UDF instances per node as there are cores
- **Formula**: MAX UDF RAM × Number of Cores × Number of Nodes
- Example: 500 MiB per UDF instance × 72 cores × 8 nodes = **282 GiB**
- Use [UDF instance limiting](https://docs.exasol.com/db/latest/database_concepts/udf_scripts/udf_instance_limit.htm) to control instances

#### 4. Additional Software

Account for monitoring services and other processes:
- Examples: Monitoring agents, log collectors, security tools
- Refer to vendor documentation for RAM requirements

### Database RAM Formula

```
Required DB RAM = 
  MAX(
    Compressed Data Volume × Estimated DB RAM %,
    Index Volume × Index Scale Factor
  )
  + Compressed Data Volume × Temp DB RAM Headroom %
  + (Max UDF RAM × Number of Cores × Number of Nodes)
```

**Parameters:**

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| **Compressed Data Volume** | Compressed size of raw data | Raw data ÷ 2.5 |
| **Estimated DB RAM %** | Percentage of compressed data for DB RAM | **20%** |
| **Index Volume** | Estimated index size | **15% of compressed** |
| **Index Scale Factor** | Headroom multiplier for index processing | **1.3** |
| **Temp DB RAM Headroom %** | Additional RAM for temp data | **5% of compressed** |
| **Max UDF RAM** | Total RAM needed for UDF instances | Depends on workload |
| **Number of Cores** | CPU cores per node | Hardware-dependent |
| **Number of Nodes** | Nodes in cluster | Configuration-dependent |

### Example Calculation

**Input Parameters:**

| Parameter | Value |
|-----------|-------|
| Raw (uncompressed) data volume | **2500 GiB** |
| Compressed data | **1000 GiB** (2500 ÷ 2.5) |
| DB RAM size as % of compressed data volume | **20%** |
| Index volume | **150 GiB** (15% of compressed) |
| Index scale factor | **1.3** |
| Headroom for temporary DB RAM | **5%** |
| Estimated UDF RAM requirement | **None** |

**Calculation Steps:**

```
Step 1: Calculate base DB RAM options
  Option A: Compressed data × 20% = 1000 GiB × 20% = 200 GiB
  Option B: Index volume × scale factor = 150 GiB × 1.3 = 195 GiB
  
  MAX(200 GiB, 195 GiB) = 200 GiB

Step 2: Add temp DB RAM headroom
  Temp headroom = 1000 GiB × 5% = 50 GiB

Step 3: Calculate total
  Required DB RAM = 200 GiB + 50 GiB = 250 GiB
```

**Explanation:**
- Required DB RAM based on compressed data volume is our starting point (200 GiB vs. 195 GiB)
- Additional headroom for TEMP DB RAM: 50 GiB (5% of 1000 GiB)
- **Total required DB RAM**: 200 GiB + 50 GiB = **250 GiB**

### Using System Tables for Running Systems

For existing Exasol systems, use statistical system tables for precise recommendations:

```sql
-- Check recommended DB RAM size
SELECT 
  RECOMMENDED_DB_RAM_SIZE_LAST_DAY,
  RECOMMENDED_DB_RAM_SIZE_LAST_WEEK,
  RECOMMENDED_DB_RAM_SIZE_LAST_MONTH
FROM EXA_DB_SIZE_LAST_DAY;

-- Check actual index size
SELECT 
  AUXILIARY_SIZE_LAST_DAY
FROM EXA_DB_SIZE_LAST_DAY;
```

See [Statistical System Tables](https://docs.exasol.com/db/latest/sql_references/system_tables/statistical_system_tables.htm) for more information.

---

## Complete Sizing Example (On-Premise)

### Scenario

Planning an on-premise Exasol cluster with the following requirements:
- **Raw data volume**: 2500 GiB
- **Redundancy**: 2
- **Number of nodes**: 4 active nodes
- **Reserve nodes**: 1
- **Backup strategy**: Local backups
  - Full backups: 2
  - Incremental backups: 3
  - Backup redundancy: 2
- **UDF usage**: Minimal
- **Business continuity**: None (no disaster recovery site)

### Step 1: Calculate Database Disk Space

| Component | Calculation | Result |
|-----------|-------------|--------|
| Compressed data | 2500 GiB ÷ 2.5 | 1000 GiB |
| Index volume (15%) | 1000 × 0.15 | 150 GiB |
| Statistical data (5%) | 1000 × 0.05 | 50 GiB |
| Net total | 1000 + 150 + 50 | 1200 GiB |
| With redundancy 2 | 1200 × 2 | 2400 GiB |
| Temp/frag headroom (60%) | 1000 × 0.60 | 720 GiB |
| **Total database disk space** | 2400 + 720 | **3200 GiB** |
| **Database disk per node** | 3200 ÷ 4 nodes | **800 GiB/node** |

### Step 2: Calculate Backup Disk Space

| Component | Calculation | Result |
|-----------|-------------|--------|
| Total data volume (net) | From above | 1200 GiB |
| Full backup data size | 1200 × 2 backups | 2400 GiB |
| Incremental backup data size | 1200 × 3 backups | 3600 GiB |
| Backup space (without redundancy) | 2400 + 3600 | 6000 GiB |
| **Backup space (with redundancy 2)** | 6000 × 2 | **12000 GiB** |
| **Backup disk per node** | 12000 ÷ 4 nodes | **3000 GiB/node** |

### Step 3: Calculate Total Disk Space per Node

| Component | Per Node |
|-----------|----------|
| Database disk space | 800 GiB |
| Backup disk space | 3000 GiB |
| **Total disk per active node** | **3800 GiB** |
| **Total disk per reserve node** | **3800 GiB** (same as active) |

### Step 4: Calculate Database RAM

| Component | Calculation | Result |
|-----------|-------------|--------|
| Base DB RAM (20%) | 1000 × 0.20 | 200 GiB |
| Index headroom | 150 × 1.3 | 195 GiB |
| Choose MAX | MAX(200, 195) | 200 GiB |
| Temp headroom (5%) | 1000 × 0.05 | 50 GiB |
| **Total DB RAM** | 200 + 50 | **250 GiB** |
| **DB RAM per node** | 250 ÷ 4 | **62.5 GiB/node** |

### Step 5: Calculate Total Node RAM

| Component | Per Node | Total Cluster |
|-----------|----------|---------------|
| DB RAM | 62.5 GiB | 250 GiB |
| OS overhead (10%) | 7 GiB | 28 GiB |
| **Required RAM/node** | **~70 GiB** | **280 GiB** |

### Final Hardware Requirements

**Per Active Node:**
- RAM: **70+ GiB** (recommend 128 GiB for headroom)
- Disk: **3800 GiB** (recommend 4 TB)
- Cores: Based on performance requirements

**Reserve Node:**
- **Same hardware** as active nodes
- RAM: 128 GiB
- Disk: 4 TB
- Cores: Match active nodes

**Total Cluster:**
- **5 nodes** (4 active + 1 reserve)
- **Total RAM**: 640 GiB (128 GiB × 5)
- **Total disk**: 20 TB (4 TB × 5)
- **DB RAM allocation**: 250 GiB

---

## Best Practices

### General Guidelines

1. **Reserve 10% RAM for OS** - Critical for system stability
2. **Use redundancy 2** - Essential for production environments
3. **Plan for growth** - Size for 12-18 months of data growth
4. **Monitor and adjust** - Use system tables to refine sizing over time
5. **Consider business continuity** - Plan for disaster recovery infrastructure

### Performance Optimization

1. **Increase DB RAM** beyond 10% minimum for better performance
2. **Monitor temp volume usage** - High usage indicates insufficient DB RAM
3. **Review index sizes** regularly - High index volumes may indicate schema issues
4. **Consider UDF RAM** carefully - Can significantly impact total RAM requirements
5. **Allocate 20% of compressed data** as DB RAM for optimal performance

### Capacity Planning

1. **Local backups** - Account for significant storage overhead
2. **Remote backups** - Preferred for production to save local storage
3. **Reserve nodes** - Must match active node specifications exactly
4. **Business continuity** - DR sites double infrastructure costs
5. **Compression assumptions** - Validate compression ratio with test data

### Common Pitfalls

❌ **Don't**:
- Allocate all node RAM to DB (reserve 10% for OS)
- Underestimate backup storage for local backups
- Use different hardware for reserve nodes
- Ignore fragmentation headroom
- Forget to account for business continuity infrastructure

✅ **Do**:
- Validate sizing with system tables on test systems
- Plan for data growth and business continuity
- Monitor performance metrics regularly
- Adjust sizing based on actual workload patterns
- Include backup storage in total disk calculations

---

## Comparison: On-Premise vs. AWS Cloud

| Aspect | On-Premise | AWS Cloud |
|--------|------------|-----------|
| **Hardware Selection** | Flexible, any compatible hardware | Limited to EC2 instance types |
| **Storage** | Any enterprise storage solution | Instance store volumes (mandatory 'd') |
| **Business Continuity** | Manual DR site setup | Multi-AZ, multi-region options |
| **Backup Storage** | Local or remote (customer choice) | S3, EBS, or local instance store |
| **Processor** | Any compatible CPU | AMD or Intel only (no ARM/Graviton) |
| **Scaling** | Manual hardware procurement | Instance resizing available |
| **Cost Model** | CAPEX (hardware purchase) | OPEX (pay-as-you-go) |

For AWS-specific sizing guidelines, see [AWS Sizing Guidelines](exasol_sizing_guidelines.md).

---

## Related Documentation

- [Fail Safety (On-Prem)](https://docs.exasol.com/db/latest/planning/fail_safety/fail_safety_on_premise.htm)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [Business Continuity](https://docs.exasol.com/db/latest/planning/business_continuity.htm)
- [Statistical System Tables](https://docs.exasol.com/db/latest/sql_references/system_tables/statistical_system_tables.htm)
- [UDF Instance Limiting](https://docs.exasol.com/db/latest/database_concepts/udf_scripts/udf_instance_limit.htm)
- [AWS Sizing Guidelines](exasol_sizing_guidelines.md) (for cloud deployments)

## Common Questions

- How much disk space do I need for my on-premise Exasol database?
- How much RAM should I allocate for Exasol DB RAM on-premise?
- What is the typical compression ratio for Exasol?
- How do I calculate required storage for local backups?
- How much storage overhead do local backups with redundancy 2 require?
- What hardware specifications are needed for reserve nodes?
- How do I size for business continuity and disaster recovery?
- How do I size for UDF workloads?
- What percentage of compressed data should be allocated as DB RAM?
- How much headroom should I plan for temporary data and fragmentation?
- How do I check actual sizing requirements on a running system?
- What is the difference between on-premise and AWS sizing?

## Summary

Proper sizing of an on-premise Exasol cluster involves:
- ✅ **Database disk space**: Compressed data + indexes + redundancy + 60% headroom
- ✅ **Backup disk space**: Full + incremental backups × redundancy (if local)
- ✅ **DB RAM calculation**: 10-20% of compressed data + temp headroom + UDF RAM
- ✅ **OS RAM reservation**: 10% of total node RAM
- ✅ **Reserve nodes**: Same hardware as active nodes for high availability
- ✅ **Business continuity**: Plan for DR infrastructure if required

**Key formulas**:
1. **Database Disk**: (Compressed + Index + Stats) × Redundancy + 60% Headroom
2. **Backup Disk (local)**: ((Full × (N+1)) + (Incremental × N)) × 2
3. **DB RAM**: MAX(20% Compressed, Index × 1.3) + 5% Temp Headroom + UDF RAM
4. **Node RAM**: DB RAM ÷ Nodes + 10% OS Overhead

**Total disk per node** = Database disk + Backup disk (if using local backups)

Use these guidelines as starting points and refine based on actual workload measurements from system tables.
