---
tool_name: c4
doc_type: guide
category: c4 Usage
title: "c4 Basic Usage Guide"
summary: "<path_to_c4_binary>/c4 <command> [options]"
---
# c4 Basic Usage Guide

## Running c4 Commands

**Syntax**:
```bash
<path_to_c4_binary>/c4 <command> [options]
```

**Examples**:
```bash
# With full path
./c4 ps

# If c4 is in PATH
c4 ps
```

## Getting Help

### General Help

```bash
c4 --help
```

### Command-Specific Help

```bash
c4 <command> --help
```

**Examples**:
```bash
c4 ps --help
c4 connect --help
c4 update --help
```

For a complete command and flag listing, see [c4 Command and Flag Reference](c4_command_reference.md).

## Common Commands Overview

| Command | Purpose | Example |
|---------|---------|---------|
| `c4 ps` | List all deployments | `c4 ps` |
| `c4 play` | Create deployment | `c4 aws play -i config.yaml` |
| `c4 connect` | Connect to deployment | `c4 connect -i PLAY_ID` |
| `c4 up` | Start nodes | `c4 up PLAY_ID` |
| `c4 down` | Stop nodes | `c4 down PLAY_ID` |
| `c4 rm` | Remove deployment | `c4 rm PLAY_ID` |
| `c4 update` | Update cluster | `c4 update cluster -p PLAY_ID -t VERSION` |
| `c4 config` | View/search parameters | `c4 config -K keyword` |

## Command Categories

### Deployment Management
- `c4 play` - Create new deployment
- `c4 ps` - List deployments
- `c4 rm` - Remove deployment

### Node Management
- `c4 up` - Start nodes
- `c4 down` - Stop nodes

### Connection
- `c4 connect` - Connect to deployment subsystems

### Maintenance
- `c4 update` - Update cluster software

### Configuration
- `c4 config` - View/search configuration parameters

## Quick Reference

### Check Deployment Status
```bash
c4 ps
```

### Create Deployment
```bash
c4 aws play -i config.yaml
```

### Connect to Database
```bash
c4 connect -i PLAY_ID
```

### Connect to COS
```bash
c4 connect -i PLAY_ID -s cos
```

### Start Stopped Nodes
```bash
c4 up PLAY_ID
```

### Stop Nodes (after stopping database!)
```bash
c4 down PLAY_ID
```

### Remove Deployment
```bash
c4 rm PLAY_ID
```

### Update Cluster
```bash
c4 update cluster -p PLAY_ID -t @exasol-VERSION
```

## Offline Deployment Note

C4 can deploy Exasol without internet connection when required Exasol artifacts are already available locally in the user home directory. In that case, c4 uses local files instead of fetching from remote sources.

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Installation](c4_installation.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Creating Deployments](c4_creating_deployments.md)
- [c4 Monitoring Deployments](c4_monitoring.md)
- [c4 Connecting to Deployments](c4_connecting.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [c4 Command and Flag Reference](c4_command_reference.md)
