---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Nodes are not booting after update to 6.1.6 in AWS"
summary: "**Symptoms:**"
---
# Nodes are not booting after update to 6.1.6 in AWS

## Problems

### . MAC-addresses could not be found

```
2019-09-05 18:49:49.323883    Error    n21    Public network interface (MAC address 00:16:3E:5D:25:A6) was not found. Found no more usable unconfigured interfaces.
2019-09-05 18:49:49.193616    Information    Booting    Start boot process stage 2 for '10.150.210.12'.
2019-09-05 18:49:49.075434    Information    n21    client mac adress of private0 () does not match the expected address (00:16:3E:70:C8:93), ignore this error
2019-09-05 18:49:48.760502    Information    n21    Initialize boot process.
2019-09-05 18:49:48.610138    Information    n21    client mac adress is '00:16:3E:70:C8:93'
2019-09-05 18:49:48.455784    Information    n21    client version is '6.1.3'
2019-09-05 18:49:48.286333    Information    n21    client ID is '10.150.210.21'
2019-09-05 18:49:48.072874    Information    Booting    Start boot process stage 2 for '10.150.210.21'.
```
**Symptoms:**

-Node is not booting

- EXAoperation is showing the following message:

```
 Public network interface (MAC address 00:16:3E:5D:25:A6) was not found. Found no more usable unconfigured interfaces.
```
- at RAM-disk you will see a second interface with different MAC (no issue with this)

```
 cat /sys/class/net/public0/address
```
Gives the correct output, but node still can't boot

### . HDD mount failed

**Symptoms:**

- rssh n$node was not possible - but ssh n$node -p2222 is fine

- when logging to ssh the message below appears:

```
sed: can't read /etc/hddmount_gpt.sh: No such file or directory
sed: can't read /etc/hddinit_gpt.sh: No such file or directory
```
## REASON

Automatic SSH-key deployment for the nodes was not working

## Resolution

### . MAC-addresses could not be found

```
$ diff /usr/opt/EXASuite-6/EXAClusterOS-6.1.6/sbin/cos-install-stage2 /usr/opt/EXASuite-6/EXAClusterOS-6.1.3/sbin/cos-install-stage2

339,343c339
<                 if [ "X${NO_MAC_CHECK}" != "Xyes" ]; then
<                     die "Public network interface (MAC address $MAC_ADDR_ETH1) was not found. Found no more usable unconfigured interfaces."
<                 else
<                     pInfo "Public network interface (MAC address $MAC_ADDR_ETH1) was not found. Found no more usable unconfigured interfaces, ignore this error"
<                 fi
---
>                 die "Public network interface (MAC address $MAC_ADDR_ETH1) was not found. Found no more usable unconfigured interfaces."
350,351d345
<     sshcmd 10 "stat /exasol/initialized >&/dev/null" && die "Do not start new boot process - another boot process already ran successfully."
<
623c617
```

### . HDD mount failed

```
for i in {11..XX}; do ssh n$i -p2222 bash /exasol/exawolke-update & done
```
