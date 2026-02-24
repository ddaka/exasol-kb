---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "INTERNAL: setup Java for iDRAC virtual console (legacy)"
summary: "Legacy procedure to launch old iDRAC virtual consoles that require Java Web Start and Firefox 38."
---
# INTERNAL: setup Java for iDRAC virtual console (legacy)

## Purpose

Use this procedure when an older iDRAC firmware requires legacy Java Web Start for the remote virtual console.

## Scope

This is a legacy runbook for old firmware generations only. Prefer modern iDRAC HTML5 console if available.

## Prerequisites

- Access to a support host with network access to iDRAC.
- Access to internal package share (`/nfs/transfer`) for legacy Firefox/JRE archives.
- A local working directory in your home path.

## Procedure (legacy Java 6 flow)

1. Reset local browser/Java profile data:

```bash
rm -rf ~/.mozilla ~/.java
```

2. Locate and copy required archives from NFS:

```bash
find /nfs/transfer -type f -iname '*firefox*'
find /nfs/transfer -type f -iname '*jre-1.6.0*'

cp /nfs/transfer/ALL/srv/20161109-132101/firefox-38.0_64.tar.bz2 .
cp /nfs/transfer/ALL/srv/20161028-124601/jre1.6.0_22.tar.gz .
```

3. Extract both archives:

```bash
tar xjf firefox-38.0_64.tar.bz2
tar xzf jre1.6.0_22.tar.gz
```

4. Open Java Control Panel:

```bash
./jre1.6.0_22/bin/jcontrol
```

5. In `jcontrol`, configure:
- Enable mixed code.
- Enable blacklist revocation checks for TLS certificates.
- Set the standalone Firefox binary as the default browser.

6. Start Firefox, open iDRAC, and click `Launch` for virtual console.

7. In the browser open dialog, choose custom program:

```text
/home/$USER/jre1.6.0_22/bin/javaws
```

8. Confirm the virtual console starts.

## Firmware-specific variant (iDRAC 2.41.40.40)

For this firmware family, use `jre1.8.0_65`:

```bash
cp "$(find /nfs/transfer -type f -iname '*firefox-38.0_64.tar.bz*' 2>/dev/null | head -1)" .
bunzip2 firefox-38.0_64.tar.bz2
tar xf firefox-38.0_64.tar

cp "$(find /nfs/transfer -type f -iname '*jre-8u*' 2>/dev/null | head -1)" .
tar xf jre-8u65-linux-x64.tar.gz
./jre1.8.0_65/bin/jcontrol
```

## Notes

- Keep legacy Java versions isolated to support hosts only.
- Do not reuse this setup for general browsing.
