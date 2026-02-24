---
tool_name: cos
doc_type: guide
category: system
title: "About /etc/rc.local and /etc/rc.local_cos"
summary: "Difference between host startup hooks and COS startup hooks, with operational guidance for Exasol environments."
---
# About /etc/rc.local and /etc/rc.local_cos

## Purpose

Explain where startup scripts run and how to safely use them in Exasol environments.

## `/etc/rc.local`

- Runs on host OS (outside COS).
- Under `systemd`, compatibility service may need explicit enablement.

Typical lifecycle commands:

```shell
sudo systemctl status rc-local.service
sudo systemctl enable rc-local.service
sudo systemctl start rc-local.service
```

## `/etc/rc.local_cos`

- Runs inside COS container context.
- Used for COS-scoped startup actions (for example specific network setup).

Edit from license/management node, then sync to all nodes:

```shell
sudo vim /etc/rc.local_cos
cos_sync_files /etc/rc.local_cos
```

## Operational notes

- Keep startup scripts minimal and deterministic.
- Test changes in non-production first.
- Revalidate after upgrades/reboots.

## Reference

- `systemd-rc-local-generator` documentation.


