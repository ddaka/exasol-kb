---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - Firmware update on Dell servers"
summary: "Internal workflow to update Dell server firmware via iDRAC and virtual media boot from an ISO shared by the license server."
---
# Internal - Firmware update on Dell servers

## Purpose

Perform controlled firmware updates on Dell servers.

## Prerequisites

- Required firmware ISO/package.
- iDRAC connectivity.
- `racadm` access.
- Plan to update iDRAC firmware first.

## 1) Extract iDRAC firmware from ISO

```shell
mount -o loop -t iso9660 linuxIso.iso /mnt
find /mnt -iname 'ESM*.BIN'
mkdir -p /var/tmp/idrac_fw
"/mnt/drm_files/repository/System Bundle (Linux)PER710 v520/ESM_Firmware_G6N28_LN32_1.97_A00.BIN" --extract /var/tmp/idrac_fw
```

If web UI is available, upload extracted `firmimg.d6` / `firmimg.d7` directly there.

Alternative via `racadm` + TFTP:

```shell
cp -vi "$(find /var/tmp/idrac_fw -iname 'firmimg.d*' | head -n1)" "$COS_DIRECTORY/var/clients/tftpboot"
ssh i11
racadm fwupdate -g -u -a <license_server_ip>
```

## 2) Prepare temporary SMB share on license server

```shell
mkdir -p /srv/smb
cp yourFirmwareUpdate.iso /srv/smb

service smb stop
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
cp ~/smb.conf.new /etc/samba/smb.conf
service smb start
smbpasswd -a root
```

Example `smb.conf.new`:

```ini
[global]
 workgroup = MYGROUP
 server string = Samba Server Version %v
 log file = /var/log/samba/log.%m
 max log size = 50
 security = user
 passdb backend = tdbsam

[smb]
 comment = SMB Directory for Maintenance
 browsable = yes
 writeable = no
 valid users = root
 path = /srv/smb
```

## 3) Trigger firmware update via virtual media

```shell
ssh i11
racadm config -g cfgRacVirtual -o cfgVirMediaAttached 1
racadm remoteimage -c -u root -p <password> -l //<license_server_ip>/smb/yourFirmwareUpdate.iso
racadm remoteimage -s
racadm config -g cfgServerInfo -o cfgServerFirstBootDevice VCD-DVD
racadm config -g cfgServerInfo -o cfgServerBootOnce 1
racadm serveraction hardreset
console com2
```

Use environment-specific secure credentials, not defaults.

## 4) Cleanup and restore Samba config

```shell
service smb stop
mv -v /etc/samba/smb.conf.bak /etc/samba/smb.conf
```

## Reference

- `internal-dell-firmware-update-using-suu-64-bit.md`


