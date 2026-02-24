---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Offline Ubuntu Update Using Docker"
summary: "This guide explains how to update an offline Ubuntu system using Docker to create an online environment for downloading packages. The `apt-offline` tool enables you to gather..."
---
# Offline Ubuntu Update Using Docker

## Overview

This guide explains how to update an offline Ubuntu system using Docker to create an online environment for downloading packages. The `apt-offline` tool enables you to gather updates on an internet-connected system and install them on an air-gapped system.

## Prerequisites

- Docker installed on a system with internet access
- SSH access to the offline Ubuntu system (or physical access)
- Both systems should ideally run the same Ubuntu version (e.g., Ubuntu 22.04)

## Architecture

- **Online System**: Docker container running Ubuntu with internet access (used for downloading packages)
- **Offline System**: Your target Ubuntu system without internet connectivity

## Procedure

### Step 1: Create and Start Docker Container (Online System)

Launch an Ubuntu Docker container with internet access and the correct architecture:

```bash
# Use --platform linux/amd64 for x86_64 systems
# Match the Ubuntu version to your offline system
docker run -dit --platform linux/amd64 \
  --name ubuntu-update-helper \
  -v ~/ubuntu-packages:/packages \
  ubuntu:22.04 /bin/bash
```

**Important:** Use `--platform linux/amd64` to ensure packages are compatible with standard x86_64 Linux systems, even if you're running Docker on ARM (e.g., Apple Silicon).

This creates a container with a volume mounted to `/packages` for easy file transfer.

### Step 2: Install apt-offline in Docker Container

Install apt-offline and download the installation packages:

```bash
# Update and install in the container
docker exec ubuntu-update-helper bash -c "apt update && apt install -y apt-offline"

# Download .deb packages for offline installation
docker exec ubuntu-update-helper bash -c "mkdir -p /packages/debs && cd /packages/debs && apt-get download python3-magic apt-offline"
```

**Example output:**

```text
Get:1 http://archive.ubuntu.com/ubuntu jammy/main amd64 python3-magic all 2:0.4.24-2 [12.6 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy/universe amd64 apt-offline all 1.8.4-1 [51.9 kB]
Fetched 64.5 kB in 1s (116 kB/s)
```

### Step 3: Copy apt-offline Packages to Offline System

From your host machine (exit Docker container or open another terminal):

```bash
# Copy the .deb packages to the offline system
scp ~/ubuntu-packages/debs/*.deb user@offline-system:/tmp/
```

### Step 4: Install apt-offline on Offline System

On the offline Ubuntu system:

```bash
sudo dpkg -i /tmp/*.deb
```

**Example output:**

```text
Selecting previously unselected package apt-offline.
(Reading database ... 111100 files and directories currently installed.)
Preparing to unpack .../apt-offline_1.8.4-1_all.deb ...
Unpacking apt-offline (1.8.4-1) ...
Setting up python3-magic (2:0.4.24-2) ...
Setting up apt-offline (1.8.4-1) ...
```

### Step 5: Generate Update Signature on Offline System

On the offline system, create a signature file that describes needed updates:

```bash
# Generate signature for updates
sudo apt-offline set /tmp/update.sig --update

# Copy signature file back to the online system
scp /tmp/update.sig user@online-system:~/ubuntu-packages/
```

**Example output:**

```text
Gathering details needed for 'update' operation
```

### Step 6: Download Updates in Docker Container

Return to the Docker container (or start it if stopped):

```bash
# If container is stopped, restart it
docker start -ai ubuntu-update-helper

# Inside the container, download updates
mkdir -p /packages/update
apt-offline get -d /packages/update /packages/update.sig
```

**Example output:**

```text
156 / 195 items: [#############################  ]  91.8%
252 / 252 items: [##############################] 100.0% of 43 MiB
Downloaded data to /packages/update
```

### Step 7: Verify Disk Space Before Transfer

Before transferring files, check available space and package size:

```bash
# Check size of downloaded updates
du -sh ~/ubuntu-packages/update

# Check available space on offline system
ssh user@offline-system "df -h /tmp"
```

**Example output:**

```text
44M    /Users/username/ubuntu-packages/update

Filesystem      Size  Used Avail Use% Mounted on
tmpfs           7.8G  1.2G  6.6G  16% /tmp
```

Ensure the offline system has at least 2x the package size available in `/tmp` to accommodate extraction and installation.

### Step 8: Transfer Updates to Offline System

From your host machine:

```bash
# Copy downloaded packages to offline system
scp -r ~/ubuntu-packages/update user@offline-system:/tmp/
```

### Step 9: Install Updates on Offline System

On the offline system:

```bash
# Install the downloaded updates
sudo apt-offline install /tmp/update
```

### Step 10: Generate Upgrade Signature on Offline System

On the offline system, create a signature for upgrades:

```bash
# Generate signature for upgrades
sudo apt-offline set /tmp/upgrade.sig --upgrade

# Copy signature file back to online system
scp /tmp/upgrade.sig user@online-system:~/ubuntu-packages/
```

**Example output:**

```text
Gathering details needed for 'upgrade' operation
```

### Step 11: Download Upgrades in Docker Container

In the Docker container:

```bash
# Download upgrade packages
mkdir -p /packages/upgrade
apt-offline get -d /packages/upgrade /packages/upgrade.sig
```

### Step 12: Verify Disk Space Before Transfer

Check space again before transferring upgrade packages:

```bash
# Check size of upgrade packages
du -sh ~/ubuntu-packages/upgrade

# Verify available space on offline system
ssh user@offline-system "df -h /tmp"
```

### Step 13: Transfer Upgrades to Offline System

From your host machine:

```bash
# Copy upgrade packages to offline system
scp -r ~/ubuntu-packages/upgrade user@offline-system:/tmp/
```

### Step 14: Install Upgrades on Offline System

On the offline system:

```bash
# Install the upgrades
sudo apt-offline install /tmp/upgrade

# Complete the upgrade process
sudo apt upgrade -m --no-download
```

## Cleanup

### On Offline System

```bash
# Remove temporary files
sudo rm -rf /tmp/update /tmp/upgrade /tmp/*.sig /tmp/*.deb
```

### On Online System

```bash
# Stop and remove Docker container
docker stop ubuntu-update-helper
docker rm ubuntu-update-helper

# Optionally remove downloaded packages
rm -rf ~/ubuntu-packages
```

## Alternative: Without SSH Access

If you don't have SSH access to the offline system, use USB drives or other media:

```bash
# Copy files from host to USB drive
cp -r ~/ubuntu-packages/debs /media/usb/
cp -r ~/ubuntu-packages/update /media/usb/
cp -r ~/ubuntu-packages/upgrade /media/usb/

# Then physically transfer the USB drive to the offline system
```

## Troubleshooting

### Docker Container Cannot Access Internet

Verify Docker's DNS settings:

```bash
docker run --rm ubuntu:22.04 ping -c 3 google.com
```

If DNS fails, restart Docker daemon or check network settings.

### Package Version Mismatch

Ensure both systems run the same Ubuntu version:

```bash
# Check version
lsb_release -a
```

### Insufficient Space

Check available space before downloading:

```bash
# On offline system
df -h /tmp

# In Docker container
df -h /packages
```

## Best Practices

1. **Version Consistency**: Always use the same Ubuntu version and architecture in the Docker container as the offline system
2. **Regular Updates**: Schedule regular update cycles to minimize security risks
3. **Signature Validation**: Keep signature files organized with dates (e.g., `update-2026-01-23.sig`)
4. **Backup First**: Create system backups before applying updates
5. **Test Updates**: If possible, test updates in a non-production environment first

## Security Considerations

- Transfer files securely (SSH/SCP with key authentication)
- Verify package integrity using signature files
- Store downloaded packages in secure locations
- Clean up temporary files after installation
