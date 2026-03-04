---
tool_name: c4
doc_type: guide
category: c4 Monitoring
title: "c4 Monitoring Deployments"
summary: "Use `c4 ps` to view all current deployments:"
---
# c4 Monitoring Deployments

## List All Deployments

Use `c4 ps` to view all current deployments:

```bash
c4 ps
```

**Example output**:
```
  N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
┌─  1  c3275f84  10    awscf   c5d.large    8.27.0      203.0.113.10    10.0.0.10    c      running  00:23:07  +∞
│   1  c3275f84  11    awscf   c5d.2xlarge  8.27.0      203.0.113.11    10.0.0.11    d      running  00:23:05  +∞
│   1  c3275f84  12    awscf   c5d.2xlarge  8.27.0      203.0.113.12    10.0.0.12    d      running  00:23:06  +∞
│   1  c3275f84  13    awscf   c5d.2xlarge  8.27.0      203.0.113.13    10.0.0.13    d      running  00:23:06  +∞
└─  1  c3275f84  14    awscf   c5d.2xlarge  8.27.0      203.0.113.14    10.0.0.14    d      running  00:23:06  +∞
```

## Output Columns

| Column | Description |
|--------|-------------|
| **N** | Session index of the deployment |
| **PLAY_ID** | Unique identifier for the deployment (used in commands) |
| **NODE** | Node ID (if available) |
| **MEDIUM** | Medium used to create deployment<br>`awscf` = AWS CloudFormation |
| **INSTANCE** | EC2 instance type for the node |
| **DB_VERSION** | Exasol database version |
| **EXTERNAL_IP** | Public IP address |
| **INTERNAL_IP** | Private network IP address |
| **STAGE** | Current deployment stage (a, b, c, d) |
| **STATE** | Current node state (running, stopped, etc.) |
| **UPTIME** | Time elapsed since node creation |
| **TTL** | Time to live (+∞ if unlimited) |

## Deployment Stages

| Stage | Description | Services Available |
|-------|-------------|--------------------|
| **a** | Cloud resources being allocated<br>Nothing reachable yet | None |
| **a1** | Node created and reachable via SSH port 22<br>c4 service not yet reachable | SSH on port 22 |
| **b** | Host booting and running startup scripts<br>c4 service running and reachable | c4 service, SSH on port 22 |
| **b1** | Working copy and dependencies being fetched<br>SSH available on configured port | SSH on configured port |
| **c** | Cluster Operating System (COS) running and reachable | COS, SSH |
| **d** | Database running and reachable | Database, COS, SSH |

**Deployment complete**: All database nodes at stage **d** and state **running**

**Access node**: Only reaches stage **c** (database doesn't run on access node)

## Node States

| State | Description |
|-------|-------------|
| **creating** | Node is being created |
| **pending** | Node preparing to enter running state |
| **running** | Node is running at indicated stage |
| **stopped** | Node is shut down and cannot be used |
| **rollingback** | Node creation failed and is rolling back |
| **succeeded** | Previous operation succeeded<br>**Note**: May indicate failed creation was successfully rolled back |
| **failed** | Previous deployment operation failed |

## Filter by Region

By default, `c4 ps` shows deployments in the region specified in `CCC_AWS_REGION`.

**Override region** on command line:
```bash
CCC_AWS_REGION=eu-central-1 c4 ps
```

## Customize Output

Customize the information shown with `c4 ps` options:

```bash
c4 ps --help
```

## Host Diagnostics for Runtime Readiness

Use host-scoped diagnostics for installation/runtime checks:

```bash
c4 diag host -i config
```

Do not use plain `c4 diag` as the primary installation-host validation because it includes broader build/developer checks.

### Example: `c4 diag host -i config` with disk issue

- FAILED `check_disks`: data disk `/dev/vda` is not readable/writable as required for user `exasol`
- OK `check_external_dependencies`
- OK `check_internal_rootless_dependencies`
- OK `check_required_params`
- OK `check_time_sync`
- OK `check_unprivileged_userns_clone`

Example interpretation: at least one host diag check failed, so disk access must be fixed first.

## Monitoring Best Practices

**Monitor deployments regularly**
- Use `c4 ps` to check status
- Verify all nodes at stage "d"
- Check for failed states

**Watch deployment progress**
- Connect during deployment to see logs
- Use `c4 connect -i PLAY_ID` during creation

**Note PLAY_ID**
- Save PLAY_ID for future operations
- PLAY_ID is permanent, session index (N) may change

## Troubleshooting c4 ps

### Issue: Stage "a" despite successful deployment

**Symptoms**: Deployment succeeded but `c4 ps` reports stage "a"

**Causes**:
- Deployment created without external IPs (`CCC_AWS_USE_EIP=false`)
- SSH key used by c4 is invalid
- c4 cannot reach instance over SSH

**Solution**: c4 ps cannot detect actual deployment stage if SSH is unavailable. Verify deployment success through other means (AWS console, database connection).

### Issue: Timeout while waiting for instance

**Symptoms**: c4 ps times out waiting for instance response

**Cause**: Large geographical distance between system running c4 and instances

**Solution**: Increase timeout value
```bash
# In config file
CCC_USER_PS_REMOTE_TIMEOUT=10000

# Or as environment variable
CCC_USER_PS_REMOTE_TIMEOUT=10000 c4 ps
```

**Default timeout**: 3000 ms (3 seconds)

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Creating Deployments](c4_creating_deployments.md)
- [c4 Connecting to Deployments](c4_connecting.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [c4 Troubleshooting](c4_troubleshooting.md)
- [c4 Command and Flag Reference](c4_command_reference.md)
