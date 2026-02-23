---
tool_name: c4
doc_type: guide
category: c4 Updating
title: "c4 Updating Deployments"
summary: "Update an Exasol cluster to a new version using `c4 update cluster`."
---
# c4 Updating Deployments

## Update Cluster Software

Update an Exasol cluster to a new version using `c4 update cluster`.

## Syntax

```bash
c4 update cluster [-P|-p PLAY_ID] -t TARGET [-i CONFIG] [--from-file FILE]
```

## Parameters

| Parameter | Description |
|-----------|-------------|
| **-P** | Updates all clusters in current deployment |
| **-p PLAY_ID** | Updates all clusters in specified deployment |
| **-t TARGET** | Exasol package version to update to<br>Format: `@exasol-VERSION` |
| **-i CONFIG** | **(Jump host only)** Path to config file used when deployment was created<br>Omit when running from database host |
| **--from-file FILE** | **(No internet only)** Path to downloaded update package |

## Examples

### Update from Jump Host with Internet

```bash
c4 update cluster -i ./config -p c3275f84 -t @exasol-2025.1.0
```

### Update from Jump Host without Internet

```bash
c4 update cluster -i ./config -p c3275f84 -t @exasol-2025.1.0 --from-file ./exasol-2025.1.0.tar.gz
```

### Update from Database Host

```bash
c4 update cluster -p c3275f84 -t @exasol-2025.1.0 --from-file ./exasol-2025.1.0.tar.gz
```

## Requirements

- Deployment must be running Exasol 8.21.0 or newer
- c4 version must be 0.4.12 or newer
- **Database must be stopped before update**

## Update Procedure

1. **Create backup** of database
2. **Stop the database**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
   ```
3. **Verify database stopped**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
   ```
4. **Run update command**:
   ```bash
   c4 update cluster -p PLAY_ID -t @exasol-NEW_VERSION
   ```
5. **Wait for update to complete**
6. **Start the database**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: MY_DATABASE'
   ```
7. **Verify database running**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
   ```

## Best Practices

**Create backups before updates**
- Full database backup
- BucketFS files backup
- Configuration files backup

**Test in staging first**
- Update test environment before production
- Verify application compatibility

**Verify database stopped**
- Check status before update
- Ensure clean shutdown

**Monitor update progress**
- Watch for errors
- Check logs if issues occur

**Verify after update**
- Check database version
- Test database connectivity
- Verify applications work

## Troubleshooting

### Update Fails

**Check**:
- Database was properly stopped
- Sufficient disk space available
- Network connectivity
- Update package is correct version

### Update Times Out

**Solution**: Increase timeout
```bash
CCC_PLAY_TIMEOUT=50m c4 update cluster -p PLAY_ID -t @exasol-VERSION
```

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [Update Exasol Documentation](https://docs.exasol.com/db/latest/administration/on-premise/updates.htm)
- [c4 Troubleshooting](c4_troubleshooting.md)
