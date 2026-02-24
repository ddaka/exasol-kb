---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Create downtime schedule for SaaS"
summary: "Create, scope, and end Datadog downtime windows for SaaS maintenance and incident work."
---
# Create downtime schedule for SaaS

## Purpose

Create temporary downtime windows to mute Datadog alerts during planned maintenance or active incident work, so responders are not distracted by expected alarms.

## Prerequisites

- Access to SaaS Production AWS account.
- Access to management node and container.
- Database UUID when creating database-specific downtime.

Related playbooks:

- `saas-how-to-find-customer-database-uuid-and-cluster-uuid.md`
- `saas-how-to-connect-to-the-instance.md`
- `saas-how-to-connect-to-the-container.md`

## Procedure

1. Connect to the management node.
1. Connect to the relevant container.
1. Change to default BucketFS directory:

```shell
cd /exa/data/bucketfs/bfsdefault/default/
```

1. Run `maintenance.py` with the required scope.

## Command Examples

Use Python 3.

### Global downtime with explicit start and end

```shell
python3 plugins/maintenance.py -s "17-02-2022 10:30" -e "17-02-2022 11:15"
python3 plugins/maintenance.py --start-time "17-02-2022 10:30" --end-time "17-02-2022 11:15"
```

### Global downtime with current time as start

```shell
python3 plugins/maintenance.py -e "17-02-2022 11:15"
python3 plugins/maintenance.py --end-time "17-02-2022 11:15"
```

### Global downtime with default 30-minute end window

```shell
python3 plugins/maintenance.py -s "17-02-2022 10:30"
python3 plugins/maintenance.py --start-time "17-02-2022 10:30"
python3 plugins/maintenance.py
```

### Database-specific downtime

```shell
python3 plugins/maintenance.py -s "17-02-2022 10:30" -e "17-02-2022 11:15" -d "ASDQWERTY123456789"
python3 plugins/maintenance.py --start-time "17-02-2022 10:30" --end-time "17-02-2022 11:15" --db-uuid "ASDQWERTY123456789"
```

### End active downtime windows early

```shell
python3 plugins/maintenance.py -ed
python3 plugins/maintenance.py --end-downtime
```

## Validation

- Confirm expected monitors are muted for the configured window.
- Confirm monitoring resumes after downtime ends.

## Notes

- Use `python3 plugins/maintenance.py --help` for full parameter details.
- Ensure date/time values follow the format expected by the script in your environment.


