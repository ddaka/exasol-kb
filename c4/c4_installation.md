---
tool_name: c4
doc_type: guide
category: c4 Installation
title: "c4 Installation Guide"
summary: "1. **Download the c4 binary**"
---
# c4 Installation Guide

## Prerequisites

**No prerequisites** are required for installing c4 itself. However, for AWS deployments:

- **AWS CLI v2** must be installed on the computer used for creating deployments
- See [Installing AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

### Install AWS CLI (for AWS deployments)

```bash
# Follow AWS documentation for your operating system
# https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
```

## Download c4 Using Browser

1. **Download the c4 binary**
   - Visit [Exasol Download Portal](https://downloads.exasol.com/exasol-8/c4)
   - Select the appropriate version for your platform

2. **Make c4 executable**
   ```bash
   chmod +x c4
   ```

## Download c4 Using Linux Command Line

Download and set permissions in a single operation:

```bash
wget https://x-up.s3.amazonaws.com/releases/c4/linux/x86_64/<version>/c4 -O c4 && chmod +x c4
```

**Parameters:**
- `<version>`: Desired c4 version (e.g., `4.28.2`)
- `-O c4`: Overwrites any existing c4 binary in the same directory

**Example:**
```bash
wget https://x-up.s3.amazonaws.com/releases/c4/linux/x86_64/4.28.2/c4 -O c4 && chmod +x c4
```

**Important**: You **must** use `chmod +x c4` to make c4 executable for all users, otherwise the application will not run.

## Updating c4

**Automatic updates**: The c4 binary is automatically updated when you update the Exasol database. There is normally no need to manually update c4.

**Manual update**: If you want to replace a currently installed version:
1. Download the version compatible with your database version from the [Download Portal](https://downloads.exasol.com/exasol-8/c4)
2. Overwrite the existing binary

**Version compatibility**: Ensure c4 version matches your Exasol database version for optimal compatibility.

## Add c4 to PATH (Optional)

To avoid prefixing commands with the path to the c4 binary:

```bash
# Example: Add to ~/.bashrc or ~/.zshrc
export PATH=$PATH:/path/to/c4/directory

# Or move c4 to a directory already in PATH
sudo mv c4 /usr/local/bin/
```

## Verify Installation

```bash
# Check c4 is executable
./c4 --help

# Or if in PATH
c4 --help
```

## Next Steps

After installation:
1. **Create configuration file** for c4
2. **Set up authentication** for your target platform (AWS, Azure, etc.)
3. See [c4 Configuration Guide](c4_configuration.md)

## Troubleshooting

### Issue: "c4: command not found"

**Cause**: c4 not in PATH or not executable

**Solution**:
```bash
# Make executable
chmod +x c4

# Run with full path
./c4 ps

# Or add to PATH
export PATH=$PATH:/path/to/c4/directory
```

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Basic Usage](c4_basic_usage.md)
