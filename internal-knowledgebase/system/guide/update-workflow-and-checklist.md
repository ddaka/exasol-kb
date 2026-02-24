---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Update Workflow & Checklist"
summary: "Operational governance checklist for Exasol updates: planning, execution ownership, and mandatory documentation flow."
---
# Update Workflow & Checklist

## Purpose

Define mandatory update process controls and checklist handling for support-led upgrades.

## Mandatory rules

* Install only released software (see [Download Portal](https://downloads.exasol.com/))
* Update requests are processed only via a Salesforce case
* Create one Salesforce case per cluster
* Assigned Support Engineer owns planning, execution, and post-upgrade acceptance
* Validate in test environment before production (see [Update Considerations](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/update_considerations.htm))
* Backup creation time is a good proxy for worst-case restore time
* Single-node ISO is internal-only and available via:
  * `/usr/opt/LICENSE_SERVER_ARCHIV/<VERSION_NUMBER>/`
  * `https://archive.dev.exasol.com/<VERSION_NUMBER>/` (support hosts)
* Security patch warning: if nodes are online/suspended, verify update process completion before rebooting
* The update checklist **must** be used for every update and must be completed in full.

## Checklist templates

Two checklist variants exist:
- Standard updates (on-prem and common flows)
- NGA updates

Templates:
- [SharePoint Update Checklist Folder](https://exasolcom.sharepoint.com/:f:/r/sites/ORG-IPS-SystemSupport/Shared%20Documents/System%20Support/Update%20Checklist?csf=1&web=1&e=Tq2T1z)

## Checklist usage workflow

1. Create a new folder in SharePoint named with the Salesforce case number.
2. Copy the correct template into that folder and rename as `Update_Checklist_<CaseNumber>.docx`.
3. Add the SharePoint checklist link as a comment in the Salesforce case.

## Canonical references

- `documents/c4/c4_updating.md`
- `documents/c4/c4_best_practices.md`
