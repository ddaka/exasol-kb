---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "ECC Monitor & Memory Monitor"
summary: "An alert was created as follows:"
---
# ECC Monitor & Memory Monitor

## Problem

An alert was created as follows:

Case Subject: ECC monitor or Memory Monitor

Case Description:
- Memory device has exceeded the Correctable Memory Event failure rate
- ECC error counter greater X
- Unknown/error state

## Procedure

Check if the affected node is an active reserve node with
`dwad_client sys-nodes <DATABASE_NAME>;`
or in EXAoperation -> EXASolution -> &lt;DATABASE_NAME&gt;

Inform the customer about the issue. If it is an active node it needs to be swapped with an reserve node as soon as possible. Therefore, ask the customer for a downtime, swapping the nodes takes about 15 minutes.
On the agreed date for a downtime swap the node.

### None Appliacne (Software only)

Ask the customer to contact their hardware vendor in order to arrange a replacement.

### Appliance & EXACloud

For appliances and EXACloud systems we take care of informing Dell and coordinating the DIMM replacement.
In order to create a Dell case we need a TSR. Connect to the iDRAC of the affected node and collect a TSR. Afterwards, contact Dell and provide it. Then schedule a replacement date with Dell and the on-site personnel of the datacenter, for EXACloud we contact Noris.

<br />
As soon as the DIMM has been replaced check the node again.

## Additional References

[Dell TSR collection](https://www.dell.com/support/kbdoc/de-de/000126308/export-a-supportassist-collection-via-idrac9?lang=en)

[Datacenter Access Noris](https://exasol.atlassian.net/wiki/spaces/SK/pages/265453618/DataCenter+Noris+Network+-+Access)


