---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - AWS: Upgrade ENA driver from 1.5.0 to 2.1.1 (live patch)"
summary: "Internal procedure to build and install ENA 2.1.1 with DKMS on AWS hosts and regenerate initramfs for persistence."
---
# Internal - AWS: Upgrade ENA driver from 1.5.0 to 2.1.1 (live patch)

## Purpose

Upgrade ENA network driver to `2.1.1` on running AWS instances.

## Prerequisites

- Maintenance/change approval.
- Root privileges.
- Network access to GitHub package source.
- Rollback plan prepared.

## Procedure

```shell
yum install -y epel-release
yum install -y dkms

cd /tmp
curl -o ena_linux_2.1.1.tar.gz https://codeload.github.com/amzn/amzn-drivers/tar.gz/ena_linux_2.1.1
tar zxvf ena_linux_2.1.1.tar.gz
mv amzn-drivers-ena_linux_2.1.1 /usr/src/ena-2.1.1

cat <<EOF2 > /usr/src/ena-2.1.1/dkms.conf
PACKAGE_NAME="ena"
PACKAGE_VERSION="2.1.1"
AUTOINSTALL="yes"
REMAKE_INITRD="yes"
BUILT_MODULE_LOCATION[0]="kernel/linux/ena"
BUILT_MODULE_NAME[0]="ena"
DEST_MODULE_LOCATION[0]="/updates"
DEST_MODULE_NAME[0]="ena"
CLEAN="cd kernel/linux/ena; make clean"
MAKE="cd kernel/linux/ena; make BUILD_KERNEL=\${kernelver}"
EOF2

dkms add -m ena -v 2.1.1
dkms build -m ena -v 2.1.1
dkms install -m ena -v 2.1.1

dracut -f --add-drivers ena
```

## Validation

```shell
modinfo ena
```

Confirm loaded/installed module version is `2.1.1`.

## Notes

- If procedure is executed through a script file, set execute permission first:

```shell
chmod +x update_ena.sh
```

## References

- <https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enhanced-networking-ena.html#ena-adapter-driver-versions>


