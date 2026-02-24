---
tool_name: c4
doc_type: troubleshoot
category: system
title: "Restarting C4 cloud command TCP ports still listenting"
summary: "The customer restarted the `c4_cloud_command` service, but the Exasol services using the TCP ports (sockets) are still listening. This can be checked by running `netstat -tpln`"
---
# Restarting C4 cloud command TCP ports still listenting

## Problem

The customer restarted the `c4_cloud_command` service, but the Exasol services using the TCP ports (sockets) are still listening. This can be checked by running `netstat -tpln`

If the `grep '^session.*system-auth$' /etc/pam.d/sudo` returns a non-empty result it means that the C4 play script didn't run properly and missed something in the process.

Normally, the C4 installation script is designed to resolve this automatically, but for some reason, it didn’t do so in this case. The issue is traced back to a PAM configuration problem.

Example output:
![Alt text](./images/netstat_tpln.png)

## Procedure

Apply the following:

```bash
sudo su -
sed -i 's/^session.*system-auth$//' /etc/pam.d/sudo
grep ^session /etc/pam.d/system-auth | grep -v pam_systemd >> /etc/pam.d/sudo
```

After the workaround is applied, a reboot of all nodes is recommended.

## Additional References

[CentOS systemd places service subprocesses started with `sudo` in `user.slice` (instead of `system.slice`)](https://unix.stackexchange.com/questions/761524/centos-systemd-places-service-subprocesses-started-with-sudo-in-user-slice)
