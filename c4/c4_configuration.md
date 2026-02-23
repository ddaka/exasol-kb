---
tool_name: c4
doc_type: reference
category: c4 Configuration
title: "c4 Configuration Guide"
summary: "c4 uses configuration parameters to define deployment settings. Parameters can be set in configuration files or on the command line."
---
# c4 Configuration Guide

## Overview

c4 uses configuration parameters to define deployment settings. Parameters can be set in configuration files or on the command line.

## Configuration File Formats

c4 supports three configuration file formats:

1. **Shell format** (`.conf`)
2. **JSON format** (`.json`)
3. **YAML format** (`.yaml` or `.yml`)

**Default configuration file**: `config` in the c4 installation directory (loaded automatically)

## Parameter Structure

Configuration parameters are organized into **modules** that indicate context:

**Naming convention**: `CCC_<MODULE>_<PARAMETER>`

**Example modules**:
- `CCC_USER_*` - User-related parameters
- `CCC_PLAY_*` - Deployment (play) parameters
- `CCC_AWS_*` - AWS-specific parameters
- `CCC_DB_*` - Database parameters

**Example parameters**:
- `CCC_USER_PASSWORD` - User password
- `CCC_PLAY_DATABASE_NAME` - Database name
- `CCC_AWS_REGION` - AWS region

## Discovering Parameters

### List All Parameters

```bash
c4 config -F
```
Returns complete list of all parameters and their current values.

### Search for Specific Parameters

```bash
c4 config -K <string>
```

**Examples**:
```bash
# All parameters in CCC_PLAY module
c4 config -K ccc_play

# All parameters containing "key"
c4 config -K key

# All timeout-related parameters
c4 config -K timeout
```

### Get Help

```bash
c4 config --help
```

## Parameter Formats

Parameters are formatted differently depending on where they're specified:

| Location | Format | Example |
|----------|--------|---------|
| **Shell config file** | Full parameter name (UPPERCASE) | `CCC_PLAY_DATABASE_NAME=my_db` |
| **JSON/YAML config** | Exclude `CCC_<MODULE>` prefix (lowercase)<br>Must be in module subsection | `play:`<br>`  database_name: my_db` |
| **Environment variable** | Full parameter name (UPPERCASE) | `CCC_PLAY_DATABASE_NAME=my_db` |
| **Command-line (absolute)** | Replace `_` with `-` (lowercase) | `--ccc-play-database-name my_db` |
| **Command-line (relative)** | Exclude `CCC_<MODULE>` prefix<br>Replace `_` with `-` (lowercase)<br>Preceded by module name | `--database-name my_db` |

## Parameter Data Types

Each parameter has a defined data type with automatic validation:

| Type | Description | Example |
|------|-------------|---------|
| **string** | Text or numbers<br>Use quotes if contains spaces | `CCC_PLAY_ADMIN_PASSWORD=df0g98j_98Xc`<br>`CCC_USER_FULLNAME="Alice User"` |
| **int** | Integer number | `CCC_PLAY_DB_PORT=8563` |
| **bool** | Boolean (true/false) | `CCC_PLAY_ACCESS_NODE=true` |
| **list** | List of strings<br>Space-separated in env vars<br>Repeat parameter on command line | Env: `CCC_MODULE_PARAM='A "B C"'`<br>CLI: `--param=A --param='B C'` |

## Setting Parameters

Configuration values can be set in multiple ways with the following **order of precedence** (later overrides earlier):

1. **User configuration file** (default `config`)
2. **Configuration file loaded on command line** (`-i <file>`)
3. **Environment variables** (before command)
4. **Command-line parameters** (highest priority)

**Example precedence**:
```bash
# config file has: CCC_PLAY_DATABASE_NAME=default_db
# Environment variable overrides it:
CCC_PLAY_DATABASE_NAME=env_db c4 play ...
# But command line wins:
CCC_PLAY_DATABASE_NAME=env_db c4 play --database-name cli_db ...
# Result: database name is "cli_db"
```

## Configuration Examples

### User Configuration File

**Shell format** (`~/.c4/config`):
```bash
CCC_USER_FULLNAME="Alice User"
CCC_USER_EMAIL="alice@example.com"
CCC_PLAY_DATABASE_NAME=my_database
CCC_PLAY_DB_PORT=8563
```

**JSON format** (`config.json`):
```json
{
  "user": {
    "fullname": "Alice User",
    "email": "alice@example.com"
  },
  "play": {
    "database_name": "my_database",
    "db_port": 8563
  }
}
```

**YAML format** (`config.yaml`):
```yaml
user:
  fullname: Alice User
  email: alice@example.com
play:
  database_name: my_database
  db_port: 8563
```

### Environment Variables

```bash
# Single variable
CCC_PLAY_DATABASE_NAME=my_db c4 play ...

# Multiple variables
CCC_PLAY_DATABASE_NAME=my_db CCC_AWS_REGION=eu-central-1 c4 play ...
```

### Command-Line Parameters

**Absolute option** (can be anywhere in command):
```bash
c4 play --ccc-play-database-name my_db --ccc-db-password j8IUgef7_IJz9 ...
```

**Relative option** (preceded by module name):
```bash
c4 play --database-name my_db --db-password j8IUgef7_IJz9 ...
```

**Mixed absolute and relative**:
```bash
c4 play --database-name my_db db --password j8IUgef7_IJz9 ...
```

## Load Custom Configuration File

```bash
c4 play -i /path/to/custom/config.yaml ...
```

## Best Practices

**Use configuration files for deployments**
- Version control your configs
- Reusable and reproducible
- Easier to review and validate

**Use meaningful names**
- Database names, deployment names
- Makes identification easier

**Keep credentials secure**
- Don't commit credentials to version control
- Use environment variables for secrets
- Restrict config file permissions (600)

**Version control configurations**
- Track changes over time
- Easy rollback if needed
- Team collaboration

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Installation](c4_installation.md)
- [c4 Creating Deployments](c4_creating_deployments.md)
- [Configuration Parameters Reference](https://docs.exasol.com/db/latest/administration/aws/c4/c4_parameters.htm)
