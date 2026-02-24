---
tool_name: cos
doc_type: guide
category: system
title: "How to change CPU governor"
summary: "Set CPU governor to performance mode persistently on Exasol hosts when conservative/ondemand scaling limits CPU frequency."
---
# How to change CPU governor

## Purpose

Set CPU governor to `performance` when frequency scaling causes sustained low CPU frequencies and degraded workload performance.

## Symptoms

`Health.log` shows entries similar to:

```text
Governor is set to 'conservative', try set to performance
CPU frequency is set to 1000.x, try to set to 3900.0
```

## Prerequisites

- Root or sudo access.
- Confirm change is allowed by platform/hardware policy.

## 1) Create governor script

Create `/usr/local/bin/set-cpu-governor.sh`:

```bash
#!/bin/bash
for cpu_gov in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
do
  echo performance > "$cpu_gov"
done
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/set-cpu-governor.sh
```

## 2) Create systemd unit

Create `/etc/systemd/system/cpugovernor.service`:

```ini
[Unit]
Description=Set CPU Governor

[Service]
Type=oneshot
ExecStart=/usr/local/bin/set-cpu-governor.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable and run:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cpugovernor.service
sudo systemctl start cpugovernor.service
```

## 3) Validate

```bash
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor | sort -u
```

Expected result:

- only `performance` is returned.

## References

- <https://documentation.ubuntu.com/server/explanation/performance/perf-tune-cpupower/>
- <https://exasol.atlassian.net/browse/L3-3585>


