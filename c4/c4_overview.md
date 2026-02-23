---
tool_name: c4
doc_type: concept
category: c4 Overview
title: "Exasol Deployment Tool (c4) - Overview"
summary: "Download from [Exasol Download Portal - c4](https://downloads.exasol.com/exasol-8/c4)"
---
# Exasol Deployment Tool (c4) - Overview

## What is c4?

**Exasol Deployment Tool (c4)** is a command-line tool for creating, configuring, and managing Exasol deployments on all supported platforms. It provides comprehensive cluster lifecycle management capabilities and access to administration interfaces.

## Key Capabilities

- Create and deploy Exasol clusters
- Monitor deployment status and health
- Connect to subsystems (database, COS, host OS)
- Update Exasol software versions
- Manage cluster lifecycle (start, stop, remove)
- Access ConfD and EXAsupport interfaces
- Configure deployments via files or command line

## Supported Platforms

- **AWS** (Amazon Web Services)
- **Azure** (Microsoft Azure)
- **GCP** (Google Cloud Platform)
- **On-premise** installations

## Latest Release

Download from [Exasol Download Portal - c4](https://downloads.exasol.com/exasol-8/c4)

## Common Commands

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

## Related Documentation

- [c4 Installation Guide](c4_installation.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Deployment Monitoring](c4_monitoring.md)
- [c4 Connection Guide](c4_connecting.md)
