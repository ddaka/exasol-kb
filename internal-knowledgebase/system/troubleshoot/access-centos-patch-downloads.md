---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to get the download links for CentOS Patches"
summary: "This guide describes how to get the link to the latest CentOS patch for Exasol version 7.1."
---
# How to get the download links for CentOS Patches

This guide describes how to get the link to the latest CentOS patch for Exasol version 7.1.

## Tracking Licenses

> **Important:** If you apply a CentOS patch on a customer system, you must ensure that the number of Tuxcare CentOS Patch licenses in use is tracked in Salesforce. Check for any open cases of type "Service Request" and category "CentOS Patch Request" that are linked to the related customer account and see if the number of nodes (case field "No. of Nodes") matches the used licenses at that account. If there is a case open but the number does not match, please enter the correct value into the field

If there is no case please create a new one. To do this, please use the following field values:

* Subject: CentOS Patch Download
* Description: "This case provides the latest CentOS Patches for Exasol Version 7.1. Please keep this case open as long as you are using Exasol Version 7.1 with one of the CentOS patches."
* Type: Service Request
* Category: CentOS Patch Request
* Account: Customer account where the update takes place
* Contact: optional! Only if a certain customer contact wants to be informed about new CentOS patch releases. Please ask your contact point.
* No. of Nodes: Enter the right amount of nodes that will receive the CentOS patch (license nodes included!)
* Visible to customer: true
* Case Origin: Internal
* Status: New
* Assignee: Support Queue

## Unsubscribing

When customer migrates to DB version 8, the respective case of category "CentOS Patch Request" is not needed anymore, as CentOS patches aren't needed anymore.
Therefore, when upgrade to DB version 8 is done, the respective "CentOS Patch Request" cases in the account should be closed.
It will also unsubscribe the Contacts linked to those cases from email notifications about new CentOS patches.

## Download Patches

You can access all CentOS patches, the changelogs and the md5 checksum at

> <https://exasol.lightning.force.com/lightning/o/CentOS_Link__c/list?filterName=All>

If you are working on a support host, you can also use the internal link

> <https://archive.dev.exasol.com/os_security_updates/daily/>
