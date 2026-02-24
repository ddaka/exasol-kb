---
tool_name: cos
doc_type: troubleshoot
category: system
title: "INTERNAL - Extra Kernel Parameter during Installation"
summary: "Sometimes hardware components (especially raid controllers) might be too new for the current EXASOL CentOS plus kernel. This article shows how to workaround this problem by:"
---
# INTERNAL - Extra Kernel Parameter during Installation

Sometimes hardware components (especially raid controllers) might be too new for the current EXASOL CentOS plus kernel. This article shows how to workaround this problem by:

1. Mount ISO, before starting the installation add kernel parameter to install command

```
install myspecialkerneloption
```

2. If the mgmt node won't boot after installation, it might be necessary to add the parameter to the kernel options as well "Press ... to add kernel options".
3. In order to enable the parameter at every boot execute following command on the management node shell

```
for f in /boot/vmlinuz-* ; do grubby --update-kernel=$f --args=myspecialkerneloption ; done
```

If the data nodes need parameters as well, execute the following command on management's node shell:

```
echo "hpsa.hpsa_allow_any=1" >> /etc/cos/boot_options
cos_mkbootimg coskillall
appserverd
```
