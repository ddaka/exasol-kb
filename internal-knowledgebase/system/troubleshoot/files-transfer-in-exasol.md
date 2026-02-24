---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Files Transfer in Exasol"
summary: "The process of transfering files or logs between local computer, support hosts and development environment."
---
# Files Transfer in Exasol

## Overview

The process of transfering files or logs between local computer, support hosts and development environment.

## Prerequisites

Local copy of the `exacp.sh` script should be located preferably in `/Users/bogdan.mihai/.local/bin/exacp.sh` on MacOS laptops, or `/usr/local/bin/exacp.sh` on Linux Subsystem (WSL).

### NOTE: There are 4 directions in which the files/logs can be pushed

----

### LOCALHOST &gt; SUPPORT HOST via TRANSFERCORE using `exacp.sh`

- Initialize Kerberos token

    ```bash
    kinit bomi@CORE.EXASOL.COM
    ```

    **NOTE (Workaround):** As of Jan 2026, the Kerberos authentication isn't working while Forti ZTNA Connection is Enabled. First you need to disable it, and then connect to IPSEC EXA. After you completed the file transfer you can disconnect from IPSEC and re-enable ZTNA.

- Use `exacp.sh` to upload file to a specific Support Host (in this example I will use Sony's Support Host as the target)

    ```bash
    exacp.sh -t sony -j {SF_case} /path/to/file
    ```

- Once the file/log has been uploaded you can open the Web Browser on the Support Host and access the following link to find your file:

    ```bash
    https://transfer.core.exasol.com/SUPPORT/
    ```

----

### LOCALHOST &gt; SUBMIT01 (DEV) via `Log File Upload URL` using the web browser (mostly used by the customer)

- Open the link from `Log File Upload URL` assigned to the Salesforce Case and upload the file/log.

- The file/log will be uploaded to the correspending `Log File Location` on SUBMIT01:/srv/bugtrack/../..

----

### SUPPORT HOST &gt; SUBMIT01 (DEV) using `upload_file.sh` (mostly used by Support)

- Upload logs on SUBMIT01 for R&D investigation

    ```bash
    upload-file.py -l {SF Log File Upload URL} -f ./path/to/file                       # if file is located on the support host
    upload-file.py -l {SF Log File Upload URL} -f root@license-server:/path/to/file    # if file is located on a remote server
    ```

----

### SUBMIT01 (DEV) &gt; SUPPORT HOST using `exacp.sh`

- Upload packages required for updates from `/usr/opt/LICENSE_SERVER_ARCHIV`

    ```bash
    exacp.sh -t sony -j {SF_case} /path/to/file
    ```

- The easist way to get access to the update packages from the Support Host is to access the following link:

    ```bash
    https://archive.dev.exasol.com/
    ```

----

## Additional References

- [Transfer customer log files to developers support hosts](/Support-and-Services/transfer-customer-log-files-to-developers-support-hosts.md)
- [Support scenarios and needed resources](https://exasol.atlassian.net/wiki/spaces/SPOTSUP/pages/4555971/Support+scenarios+and+needed+resources)
- [Log Files for Support, version 7.1](https://docs.exasol.com/db/7.1/administration/on-premise/support.htm)
- [Exasol log files explained v7](Exasol-log-files-explained-v7.md)
- [Log Files for Support, version 8](https://docs.exasol.com/db/latest/administration/on-premise/support.htm)
- [Exasol log files explained v8](Exasol-log-files-explained-v8.md)
- [How to get log files from Exasol SaaS systems (temporary solution)](/SaaS/how-to-get-log-files-from-exasol-saas-systems-temporary.md)
- [Collect logs from no-container running clusters/databases on SaaS](/SaaS/collect-logs-from-no-container-running-clusters-on-saas.md)


