# Exasol Update Procedure (On-Premise)

**Category:** Administration  
**Topic:** Updates, Upgrades, Maintenance, Installation, Version Management  
**Keywords:** update, upgrade, patch, version, c4, Exasol Admin, deployment, maintenance, downtime, backup, rollback, installation  
**Source:** [Exasol Update Documentation](https://docs.exasol.com/db/latest/administration/on-premise/updates.htm)

## Overview

This document explains how to update your Exasol database to a newer version in an **on-premise deployment**. Keeping your Exasol software updated ensures optimal performance, high security, and access to the latest features.

**Key Benefits of Updates:**
- Optimal database performance
- Latest security patches
- New features and functionality
- Bug fixes and stability improvements

**Important**: Always read [Update Considerations](#update-considerations) before starting any update.

---

## Update Methods by Version

Different Exasol versions support different update methods:

| Current Version | Update Method | Tool | Notes |
|----------------|---------------|------|-------|
| **Exasol 2025.1 and later** | UI or Command Line | Exasol Admin UI or c4 | Admin UI only available for on-premises deployments |
| **Exasol 8.21.0 - 8.34.0** | Command Line | c4 (Exasol Deployment Tool) | Use `c4 update cluster` command |
| **Exasol 8.20.0 or earlier** | Migration Required | Backup & Restore | Create new deployment and restore backup |

**Note**: For versions prior to 8.21.0, you must create a new deployment and restore a backup. See [Migrate from Exasol 7.1](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/migrate_71_v8.htm).

For release information, see [Release Notes](https://docs.exasol.com/db/latest/release_notes.htm).

---

## Update Considerations

### Planning

Planning is crucial for minimizing the maintenance window and ensuring a successful update.

#### 1. Read Release Notes

**Critical**: Read release notes for all versions between your current and target version.

- **Release notes describe changes** introduced in each specific release
- **Understand impact** on your system and operations
- **Assess criticality** for your business
- **Cumulative changes**: Read all interim release notes, not just the target version

**Resource**: [Release Notes - Exasol 8 Rolling Releases](https://docs.exasol.com/db/latest/release_notes_db.htm)

For information about release types, see [Product Life Cycle](https://docs.exasol.com/db/latest/planning/life_cycle.htm).

#### 2. Schedule During Low-Impact Windows

Plan the update for a time when it will have **minimum impact on business operations**.

❌ **Avoid**:
- Close to business-critical events (product releases, quarter-end, etc.)
- Peak business hours
- During major campaigns or promotions

✅ **Recommend**:
- Scheduled maintenance windows
- Off-peak hours or weekends
- After thorough testing in test environment

**Note**: Updates always involve some downtime for database restart.

#### 3. Create Full Remote Backups

**Mandatory**: Create remote backups before ANY update.

**What to back up**:
1. **Database data**: Full remote backup
2. **BucketFS files**: UDF scripts, drivers, custom JARs, certificates

**Why remote backups are critical**:
- Allows rollback without data loss if update fails
- Local backups may be inaccessible if update procedure fails
- BucketFS files are NOT included in database backups

**Rollback procedure** (if update fails):
1. Reinstall previous Exasol version
2. Restore database from remote backup
3. Restore BucketFS files manually

**Resources**:
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm)
- [BucketFS Client](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs_client.htm)

### Testing

**Strongly recommended**: Test the new version in a test environment before updating production.

#### Testing Workflow

1. **Define Acceptance Criteria**
   - Core functionality requirements
   - Integration compatibility (drivers, tools, BI platforms)
   - Script language support (Python, R, Java, Lua)
   - Performance benchmarks

2. **Set Up Test Environment**
   - **Mirror production structure**: Same basic schema, users, roles, tables
   - **Similar hardware specs**: Match production sizing where possible
   - **Install new Exasol version** on test cluster

3. **Run Acceptance Tests**
   - **Review changes**: Test based on release note changes
   - **Simulate production load**: Mimic typical query patterns and concurrency
   - **Test frequent queries**: Use audit logs to identify most-used queries
   - **Verify UDFs**: Test all user-defined functions used in production
   - **Test integrations**: Verify all drivers, connectors, and BI tools
   - **Use latest drivers**: If new version supports updated drivers, test with latest versions

4. **Performance Validation**
   - Run performance benchmarks from production
   - Compare query execution times
   - Verify resource utilization (CPU, RAM, disk I/O)
   - Check for performance regressions

5. **Integration Testing**
   - Test all data pipelines
   - Verify ETL processes
   - Test BI tool connectivity (Tableau, Power BI, etc.)
   - Validate custom applications

**Need Help?**: [Create a support case](https://exasol.my.site.com/s/create-new-case) if you need assistance.

---

## Prerequisites

Before starting the update, ensure the following conditions are met:

### General Prerequisites

✅ **All data nodes must be online**
- Check node status before update
- All active nodes must be in "running" state

✅ **Database must be stopped**
- Use `db_stop` ConfD job
- Verify database is fully stopped before proceeding

✅ **Sufficient disk space**
- **Requirement**: At least **20 GiB free space** in home directory partition on each host
- Check: `/home/<exasol_user>/`
- Free up space if needed before update

### Additional Prerequisites for c4 Updates

When using `c4 update cluster` (command line method):

#### Version Requirements

- **Deployment version**: Exasol 8.21.0 or newer
- **c4 version**: 0.4.12 or newer
- **Tool documentation**: [Install c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/c4_install.htm)

#### System Requirements

- **SSH access**: User executing update must have SSH privileges to cluster
- **Internet access**: Required unless using `--from-file` option
- **Installation package**: If no internet, download and copy package to update system

#### Jump Host Requirements (External Update System)

When updating from a system **outside the cluster**:

✅ **SSH installed** and key-based authentication configured
✅ **c4 binary path** must be specified (e.g., `./c4 update ...`)
✅ **rsync installed** (only for c4 version 4.24.3 or earlier; not required for 4.25.0+)
✅ **Configuration file path** must be specified with `-i` option

**Configuration file**: The file used when the deployment was originally created

---

## Update Procedure (Exasol Admin UI)

**Available in**: Exasol 2025.1 and later (on-premises only)

### Method 1: Update from Cloud (Recommended)

**Available in**: Exasol 2025.2 and later

This method downloads the update directly from Exasol cloud servers.

**Steps**:

1. **Log in to Exasol Admin**
   - Access the Admin UI via browser

2. **Navigate to Databases page**
   - Click the **Update** button

   ![Admin UI Update Button](https://docs.exasol.com/db/latest/resource/images/adminui/adminui-update-start.png)

3. **Select Auto tab**
   - Choose target version from dropdown menu
   - Click **Update** to start

   ![Update Dialog - Auto](https://docs.exasol.com/db/latest/resource/images/adminui/adminui-update-auto-dialog.png)

4. **Wait for completion**
   - Update process runs automatically
   - Database restarts automatically upon successful completion

### Method 2: Update from Local File

**Available in**: Exasol 2025.1 and later

Use this method when the cluster has no internet access or you prefer manual package management.

**Steps**:

1. **Download Exasol archive**
   - Visit [Exasol Download Portal](https://downloads.exasol.com/)
   - Download the target version archive
   - Save to your local system

2. **Log in to Exasol Admin**
   - Access the Admin UI via browser

3. **Navigate to Databases page**
   - Click the **Update** button

   ![Admin UI Update Button](https://docs.exasol.com/db/latest/resource/images/adminui/adminui-update-start.png)

4. **Select Manual tab**
   - Click **Choose a file to upload**
   - Select the downloaded archive

   **Note**: In Exasol 2025.1, only the manual option is available.

   ![Update Dialog - Manual](https://docs.exasol.com/db/latest/resource/images/adminui/adminui-update-manual-dialog.png)

5. **Upload and update**
   - Click **Upload file and update**
   - Wait for upload to complete

   ![Upload Dialog](https://docs.exasol.com/db/latest/resource/images/adminui/adminui-update-manual-upload.png)

6. **Wait for completion**
   - Update process runs automatically
   - Database restarts automatically upon successful completion

---

## Update Procedure (Command Line)

**Available for**: Exasol 8.21.0 and later

Use the `c4 update cluster` command to update from the command line. You can run the update from:
- **Any host in the cluster** (database node)
- **External system (jump host)** with SSH access

### Package Management

The `c4 update cluster` command automatically manages Exasol packages on nodes:

**Retention policy**:
- Keeps **at most 2 installation packages** on each node:
  - Currently installed version
  - Previously installed version (before last update)
- **Older packages are removed** automatically
- **Script language containers** in installation directory are retained after update
- **Container cleanup**: Deleted by subsequent updates

**Location**: `~/.ccc/*` on each node

### Step 1: Get the Play ID

You need the play ID of the deployment to update. Find it using `c4 ps`.

**Command**:
```bash
c4 ps
```

**Example output**:
```
  N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
┌─  1  c3275f84  11    awscf   r5d.large    8.34.0      203.0.113.11    10.0.0.11    d      running  03:50:12  +∞
│   1  c3275f84  12    awscf   r5d.large    8.34.0      203.0.113.12    10.0.0.12    d      running  03:50:13  +∞
│   1  c3275f84  13    awscf   r5d.large    8.34.0      203.0.113.13    10.0.0.13    d      running  03:50:13  +∞
└─  1  c3275f84  14    awscf   r5d.large    8.34.0      203.0.113.14    10.0.0.14    d      running  03:50:13  +∞
```

In this example, the play ID is **c3275f84**.

### Step 2: Stop the Database

**Critical**: The database must be stopped before updating.

**Command**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
```

**Parameters**:
- `PLAY_ID`: The play ID from Step 1
- `MY_DATABASE`: Your database name

**Example**:
```bash
c4 connect -i c3275f84 -s cos -- 'confd_client db_stop db_name: PROD_DB'
```

**See also**: [How to use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm), [db_stop](https://docs.exasol.com/db/latest/confd/jobs/db_stop.htm)

### Step 3: Run Preplay Script (Rootless Installations Only)

**Required only for**: Non-root (rootless) Exasol user installations

If the Exasol user on the hosts is a non-root user, run the preplay script on each host before updating.

**Purpose**: Ensures preplay settings changed between releases are updated

**Critical**: Use the **c4 version matching the target Exasol version**, not the current version.

**Steps**:

1. **Download matching c4 version**
   - Download c4 version matching target Exasol version
   - Save to home directory of Exasol user on the host
   - See [Install c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/c4_install.htm)

2. **Run preplay script**
   - Run as root or with sudo
   - If logged in as root, omit `sudo`

   **Command**:
   ```bash
   sudo ./c4 _ preplay EXASOL_USER
   ```

   **Parameters**:
   - `EXASOL_USER`: Username of the Exasol user (non-root user running Exasol)

   **Example**:
   ```bash
   sudo ./c4 _ preplay exasol
   ```

3. **Repeat on each host**
   - Run the preplay script on all database nodes

**See also**: [Rootless Installation](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)

### Step 4: Run the Update Command

Execute the `c4 update cluster` command to perform the update.

**Syntax**:
```bash
c4 update cluster [-P|-p PLAY_ID] -t TARGET [-i CONFIG] [--from-file FILE] [--ccc-update-ccc true]
```

**Parameters**:

| Parameter | Description |
|-----------|-------------|
| **-P** | Updates all clusters in the current deployment (where command was started) |
| **-p PLAY_ID** | Updates all clusters in the deployment identified by PLAY_ID |
| **-t TARGET** | The Exasol package to use as update target (version to update to)<br>Format: `@exasol-VERSION` (e.g., `@exasol-2025.1.0`) |
| **-i CONFIG** | **(Jump host only)** Path to configuration file used when deployment was created<br>**Must be omitted** when running from database host |
| **--from-file FILE** | **(No internet only)** Path to downloaded update package<br>Required when update system has no internet access |
| **--ccc-update-ccc true** | (Optional) Updates c4 itself during the update process |

#### Example 1: Jump Host with Internet Access

```bash
c4 update cluster -i ./config -p c3275f84 -t @exasol-2025.1.0
```

**Scenario**: Updating from external jump host, cluster can access internet

#### Example 2: Jump Host without Internet Access

```bash
c4 update cluster -i ./config -p c3275f84 -t @exasol-2025.1.0 --from-file ./exasol-2025.1.0.tar.gz
```

**Scenario**: Updating from external jump host, no internet access, installation package already downloaded

#### Example 3: Database Host (Inside Cluster)

```bash
c4 update cluster -p c3275f84 -t @exasol-2025.1.0 --from-file ./exasol-2025.1.0.tar.gz
```

**Scenario**: Updating from one of the database hosts, installation package exists on the host

**Note**: `-i config` option is **omitted** when running from database host

### Verification

After the update completes, verify the installation:

1. **Check database version**
   ```sql
   SELECT PARAM_VALUE FROM EXA_METADATA WHERE PARAM_NAME = 'databaseProductVersion';
   ```

2. **Verify node status**
   ```bash
   c4 ps
   ```
   - All nodes should show the new version
   - All nodes should be in "running" state

3. **Start the database** (if not auto-started)
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: MY_DATABASE'
   ```

4. **Run smoke tests**
   - Connect to the database
   - Run basic queries
   - Verify critical integrations
   - Check UDF functionality

### Update Log

The update log provides detailed information about the update procedure.

**Location**: `/var/log/ccc/update.log` on each node

**View the log**:
```bash
c4 connect -t1/host
tail /var/log/ccc/update.log
```

**Example log output**:
```
[2023-10-27 13:01:56] Doing offline update...
[2023-10-27 13:01:59] Switched to the new version: branchr-saas-22da2bac-64r (@exasol-8.23.4)
[2023-10-27 13:01:59] Starting up...
[2023-10-27 13:01:59] Waiting for exainit is finished
[2023-10-27 13:02:00] Checking /home/ubuntu/.ccc/play/local/c3275f84-f328-42c1-8aac-49c29a8776bf/main/10/data/etc/init_done
[2023-10-27 13:03:00] Done: exainit is finished successfully
[2023-10-27 13:03:00] SUCCESS: Update was successful.
[2023-10-27 13:03:00] SUCCESS: Database is being started
[2023-10-27 13:03:00] Cleaning up local c4 packages repository
[2023-10-27 13:03:00] SUCCESS: Cleanup was successful.
```

**Key log entries**:
- **"Doing offline update"**: Update process started
- **"Switched to the new version"**: New version activated
- **"SUCCESS: Update was successful"**: Update completed successfully
- **"SUCCESS: Database is being started"**: Database restart initiated

### Usage Notes

⚠️ **Important considerations**:

1. **SSH Access Required**
   - Update procedure uses `c4 connect` internally
   - User/role starting update must have SSH access as privileged user
   - Key-based authentication recommended

2. **Version Compatibility**
   - `c4 update` command requires **Exasol 8.21.0 or later**
   - For versions **prior to 8.21.0**, use migration procedure:
     1. Create backup of old database
     2. Install latest Exasol version
     3. Restore backup on new database
   - See [Migrate from Exasol 7.1](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/migrate_71_v8.htm)

3. **Downtime**
   - Update requires database to be stopped
   - Plan for maintenance window
   - Database restarts automatically after successful update

4. **Rollback**
   - If update fails, reinstall previous version
   - Restore from remote backup
   - Restore BucketFS files manually

---

## Update Workflow Comparison

### Exasol Admin UI (2025.1+)

```
┌─────────────────────────────────────────────────────┐
│ 1. Create remote backups (DB + BucketFS)           │
├─────────────────────────────────────────────────────┤
│ 2. Stop database (automatic via UI)                │
├─────────────────────────────────────────────────────┤
│ 3. Log in to Exasol Admin                          │
├─────────────────────────────────────────────────────┤
│ 4a. Auto: Select version from dropdown             │
│     OR                                              │
│ 4b. Manual: Upload downloaded archive              │
├─────────────────────────────────────────────────────┤
│ 5. Click Update/Upload and update                  │
├─────────────────────────────────────────────────────┤
│ 6. Wait for automatic completion                   │
├─────────────────────────────────────────────────────┤
│ 7. Database restarts automatically                 │
├─────────────────────────────────────────────────────┤
│ 8. Verify installation                             │
└─────────────────────────────────────────────────────┘
```

### c4 Command Line (8.21.0+)

```
┌─────────────────────────────────────────────────────┐
│ 1. Create remote backups (DB + BucketFS)           │
├─────────────────────────────────────────────────────┤
│ 2. Get play ID (c4 ps)                             │
├─────────────────────────────────────────────────────┤
│ 3. Stop database (db_stop via c4 connect)          │
├─────────────────────────────────────────────────────┤
│ 4. [Rootless only] Run preplay script on all hosts │
├─────────────────────────────────────────────────────┤
│ 5. Run c4 update cluster command                   │
├─────────────────────────────────────────────────────┤
│ 6. Monitor update progress                         │
├─────────────────────────────────────────────────────┤
│ 7. Check update log (/var/log/ccc/update.log)      │
├─────────────────────────────────────────────────────┤
│ 8. Verify installation (c4 ps, version query)      │
├─────────────────────────────────────────────────────┤
│ 9. Start database (if not auto-started)            │
└─────────────────────────────────────────────────────┘
```

### Migration (Pre-8.21.0)

```
┌─────────────────────────────────────────────────────┐
│ 1. Create full remote backup (DB + BucketFS)       │
├─────────────────────────────────────────────────────┤
│ 2. Install new Exasol version on new cluster       │
├─────────────────────────────────────────────────────┤
│ 3. Create empty database on new cluster            │
├─────────────────────────────────────────────────────┤
│ 4. Restore database backup on new cluster          │
├─────────────────────────────────────────────────────┤
│ 5. Restore BucketFS files manually                 │
├─────────────────────────────────────────────────────┤
│ 6. Verify all data and configurations              │
├─────────────────────────────────────────────────────┤
│ 7. Test integrations and UDFs                      │
├─────────────────────────────────────────────────────┤
│ 8. Switch applications to new cluster              │
└─────────────────────────────────────────────────────┘
```

---

## Troubleshooting

### Common Issues

#### Issue: Update Fails with "Database not stopped"

**Cause**: Database is still running

**Solution**:
```bash
# Verify database is stopped
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'

# If still running, stop it
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
```

#### Issue: "Insufficient disk space"

**Cause**: Less than 20 GiB free in home directory

**Solution**:
```bash
# Check disk space on each host
c4 connect -i PLAY_ID -- 'df -h ~'

# Clean up old packages (c4 does this automatically, but can be done manually)
# Remove old logs, temporary files, or unused data
```

#### Issue: "SSH connection failed"

**Cause**: Key-based authentication not configured or SSH access denied

**Solution**:
- Verify SSH key is added to authorized_keys on all hosts
- Check SSH user has necessary privileges
- Test SSH connection manually: `ssh user@host`

#### Issue: Update log shows errors

**Cause**: Various (check specific error message)

**Solution**:
1. Review complete update log: `/var/log/ccc/update.log`
2. Check for specific error messages
3. Verify prerequisites (disk space, node status, database stopped)
4. [Create a support case](https://exasol.my.site.com/s/create-new-case) with log details

#### Issue: Database doesn't restart after update

**Cause**: Update completed but auto-start failed

**Solution**:
```bash
# Manually start the database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: MY_DATABASE'

# Check database status
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

### Rollback Procedure

If the update fails and you need to roll back:

**Prerequisites**:
- ✅ Remote backup created before update
- ✅ BucketFS files backed up to local system

**Steps**:

1. **Reinstall previous Exasol version**
   - Deploy previous version using c4 or installation method
   - Or restore previous c4 package on nodes

2. **Restore database from remote backup**
   ```bash
   # Stop current database
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
   
   # Restore backup
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_restore ...'
   ```

3. **Restore BucketFS files**
   - Manually upload backed-up files to BucketFS
   - Use BucketFS Client or REST API
   - See [BucketFS Client](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs_client.htm)

4. **Verify rollback**
   - Check database version
   - Verify data integrity
   - Test critical queries and integrations

5. **Investigate failure**
   - Review update logs
   - [Create support case](https://exasol.my.site.com/s/create-new-case) with details

---

## Best Practices

### Before Update

✅ **Read all interim release notes** between current and target versions
✅ **Create remote backups** of database and BucketFS files
✅ **Test in staging environment** that mirrors production
✅ **Schedule during low-impact windows** (off-peak hours, weekends)
✅ **Verify prerequisites** (disk space, node status, SSH access)
✅ **Download installation package** if no internet access
✅ **Notify stakeholders** of maintenance window
✅ **Prepare rollback plan** in case of failure

### During Update

✅ **Monitor update progress** via logs and output
✅ **Don't interrupt** the update process
✅ **Keep terminal session active** (use `screen` or `tmux` for remote sessions)
✅ **Save update logs** for troubleshooting if needed

### After Update

✅ **Verify version** using SQL query
✅ **Check node status** with `c4 ps`
✅ **Run smoke tests** on critical queries
✅ **Test integrations** (BI tools, ETL processes)
✅ **Verify UDFs** are working correctly
✅ **Monitor performance** for first few hours
✅ **Check update log** for warnings or errors
✅ **Update documentation** with new version number
✅ **Communicate success** to stakeholders

### General Guidelines

✅ **Always use remote backups** - Never rely on local backups during updates
✅ **Test first** - Always test updates in non-production environment
✅ **Plan for growth** - Verify disk space requirements for new version
✅ **Stay current** - Regular updates are easier than big version jumps
✅ **Document changes** - Keep update log and notes for audit trail

---

## Common Pitfalls

❌ **Don't**:
- Skip reading release notes
- Update without creating remote backup
- Update production without testing in staging
- Forget to back up BucketFS files
- Update during business-critical periods
- Interrupt the update process
- Assume local backups are sufficient
- Skip verification steps after update
- Update from version prior to 8.21.0 using c4 (use migration instead)

✅ **Do**:
- Read ALL interim release notes
- Create remote backup of database AND BucketFS
- Test thoroughly in staging environment
- Verify all prerequisites before starting
- Monitor update logs in real-time
- Have rollback plan ready
- Communicate maintenance window to users
- Verify installation after completion

---

## Quick Reference

### Update Decision Tree

```
What version are you running?
│
├─ Exasol 2025.1 or later (on-premise)
│  └─→ Use Exasol Admin UI (recommended) OR c4 command line
│
├─ Exasol 8.21.0 - 8.34.0
│  └─→ Use c4 command line (c4 update cluster)
│
└─ Exasol 8.20.0 or earlier
   └─→ Migration required (backup → new install → restore)
```

### Essential Commands

```bash
# Get play ID
c4 ps

# Stop database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: DATABASE'

# Update cluster (jump host with internet)
c4 update cluster -i ./config -p PLAY_ID -t @exasol-VERSION

# Update cluster (jump host without internet)
c4 update cluster -i ./config -p PLAY_ID -t @exasol-VERSION --from-file ./exasol-VERSION.tar.gz

# Update cluster (database host)
c4 update cluster -p PLAY_ID -t @exasol-VERSION --from-file ./exasol-VERSION.tar.gz

# View update log
c4 connect -t1/host
tail -f /var/log/ccc/update.log

# Check version after update
SELECT PARAM_VALUE FROM EXA_METADATA WHERE PARAM_NAME = 'databaseProductVersion';

# Start database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: DATABASE'
```

---

## Related Documentation

- [Release Notes - Exasol 8 Rolling Releases](https://docs.exasol.com/db/latest/release_notes_db.htm)
- [Product Life Cycle](https://docs.exasol.com/db/latest/planning/life_cycle.htm)
- [Update Considerations](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/update_considerations.htm)
- [Migrate from Exasol 7.1](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/migrate_71_v8.htm)
- [Install c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/c4_install.htm)
- [How to use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm)
- [BucketFS Client](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs_client.htm)
- [Exasol Admin UI Overview](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/admin_ui_overview.htm)
- [Rootless Installation](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)
- [db_stop ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_stop.htm)
- [db_start ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_start.htm)

## Common Questions

- How do I update Exasol to a newer version?
- What is the difference between Exasol Admin update and c4 update?
- Can I update Exasol without downtime?
- How do I update from Exasol 7.1 or 8.20.0 to the latest version?
- What prerequisites are required before updating Exasol?
- How much disk space is needed for an Exasol update?
- How do I create a remote backup before updating?
- How do I roll back an Exasol update if it fails?
- What is the c4 update cluster command syntax?
- How do I update Exasol from a jump host?
- How do I update Exasol without internet access?
- What is the preplay script and when do I need to run it?
- Where can I find the Exasol update log?
- How do I verify the update was successful?
- What should I test before updating production?
- How do I back up BucketFS files before an update?
- Can I update multiple clusters at once with c4?
- What happens to old Exasol packages after an update?

## Summary

Updating Exasol involves:
- ✅ **Planning**: Read release notes, schedule maintenance window, define acceptance criteria
- ✅ **Backup**: Create remote backup of database AND BucketFS files
- ✅ **Testing**: Test update in staging environment before production
- ✅ **Prerequisites**: Verify node status, disk space, database stopped
- ✅ **Update method**: Exasol Admin UI (2025.1+) or c4 command line (8.21.0+)
- ✅ **Verification**: Check version, node status, run smoke tests
- ✅ **Rollback plan**: Be prepared to reinstall and restore if update fails

**Update methods**:
1. **Exasol Admin UI** (2025.1+): Auto download or manual upload
2. **c4 command line** (8.21.0+): `c4 update cluster` command
3. **Migration** (pre-8.21.0): Backup → new install → restore

**Critical steps**:
1. Read ALL interim release notes
2. Create REMOTE backups (database + BucketFS)
3. Test in staging environment
4. Stop database before update
5. Run update command
6. Verify installation

**For help**: [Create a support case](https://exasol.my.site.com/s/create-new-case)
