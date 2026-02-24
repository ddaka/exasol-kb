---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Incident Runbook"
summary: "This runbook is written to help analyzing a problem in case of an incident. Also, general guidelines and links to KB article with what to do for different issues."
---
# Incident Runbook

This runbook is written to help analyzing a problem in case of an incident. Also, general guidelines and links to KB article with what to do for different issues.

## General Rules

- Provide updates at least one hour after the last customer reaction via Salesforce
- Even if there is no update "Dear customer, we’re working on this issue there is no update at the moment but we’ll keep you posted"
- Make notes of phone calls to the customer (summary mail in salesforce)
- If the issue is a known bug that is already fixed in one of the new releases tell it to the customer!
- If they cannot upgrade because of bugs in newer versions that would affect them too, provide a workaround if possible
- Check what Service Package/s a customer has! (If in doubt, help the customer + Escalate to SE)
- If there are two "Blocker" at the same time activate on-call backup
- Ask for help/escalate if you cannot fix the problem within 1 hour (contact Backup Hotline or SE)!
- If you can fix it from within EXAoperation do not use the shell
- If you cannot start working within one hour activate the backup hotline
- Check for already existing known errors and workarounds - articles in salesforce knowledge base & changelog
- Developers are available during business hours, postpone further results until any dev is available, e.g. SQL syntax debugging
- [Phone Numbers](https://exasol.atlassian.net/wiki/spaces/IT/pages/19893092/Telephone+numbers)
- IT Service +49 911 2399 112

## Customer Call via Hotline

The main goal is to let the customer know we will help him ASAP. Second aim is to get as much information as possible during the call to identify the issue.

Example:
"Database is slow"
- Who is calling?
- Did you create a case in Salesforce? Please send an email to service@exasol.com or use the webform
- Which database instance/cluster is affected?
- What priority has this incident?
- What was observed and since when does the problem exist?
- Did you try to fix it on your own if yes what did you do?
- Can you provide Session IDs? (if applicable)
Check if
- Remote Access via VPN is available
- If not, schedule a screen sharing session
- As it is an incident the screen sharing cannot take place "10 hours" later but should take place within the next hour "ASAP"

## Incident Cases

For every incident you should do a basic check of the cluster, see [How to get started during an incident](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/How-to-get-started-during-an-incident.md)

For every monitoring alert there is a KB article you can find them [here](https://github.com/exasol/internal-knowledgebase/tree/main/Monitoring-Alerts) or in salesforce the article for the alert will be recommended to you.

Other issues:

[Blackout](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/Blackout.md)
[Enlargement failed](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/Enlargement-failed.md)
[Backup slow](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/Backup-slow.md)
[License expired](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/License-expired.md)
[Database does not start](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/Database-does-not-start.md)

## Useful commands

Check storage segment distribution
`csinfo -g -i <Volume-ID>`

Show Storage recovery in percent
`csrec -s -v <Volume-ID>`

Restart Appserverd from the commandline
`coskillall appserverd`

Remove a partition from the cluster (e.g. appserverd)
`cosrm -a appserverd-partition-id`

Start new appserverd partition (Startup: tailf /var/log/logd/Appserver.log)
`cosexec --single-instance --auto-restart --auto-add – $COS_DIRECTORY/libexec/appserverd`

Recreate Boot Images for data nodes
```
cos_mkbootimg
coskillall appserverd
tailf /var/log/logd/Appserverd.log
````
