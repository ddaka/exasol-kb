---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to generate system/database statistics (SOL-520)"
summary: "Using this instruction the System Support or DB engineer could easily generate the so-called SOL-520 (database statistics for performance analysis, [Statistics export for..."
---
# How to generate system/database statistics (SOL-520)

## Overview

Using this instruction the System Support or DB engineer could easily generate the so-called SOL-520 (database statistics for performance analysis, [Statistics export for support](https://exasol.my.site.com/s/article/Statistics-export-for-support?language=en_US)).

The journey starts by connecting to the corresponding Customer's Support Host in Guacamole.

For v8 clusters, the choice of user (root or something else) follows usual rules of connecting to a v8 DB (see also [Connecting to customer DB via Support Host](/Support-and-Services/connecting-to-customer-db-via-support-host.md)).
Transferring files back and forth might also require additional `scp` steps.

1. List the available license servers.

    ```shell
    cat /etc/hosts | grep license
    ```

2. Pick the needed server and connect using the credentials from the password store ("Keeper Password Manager" as of 2023-12-06).

    ```shell
    ssh root@<lic server name>
    ```

3. Agree to add the server fingerprint to the list of known hosts.

4. Check if one of "sol520*.sh" files already exists

    ```shell
    ls -l | grep 520
    ```

5. If the file doesn't exist, create it by:

    a) Disconnect from the management node

    ```shell
    exit
    ```

    b) Get the respective sol520*.sh file

    ```shell
    # For version 7.0
    curl https://archive.dev.exasol.com/Support/SOL_520_Stats/sol520_v70.sh --output sol520_v70.sh

    # For versions 7.1 and 8
    curl https://archive.dev.exasol.com/Support/SOL_520_Stats/sol520_v71_v8.sh --output sol520_v71_v8.sh
    ```

    Note: The script `sol520_v71_v8.sh` was tested with

    * 7.1.24 Docker DB
    * 7.1.24 Community Edition (with EXAoperation)
    * 8.27.0 DB
    * 8.18.1 SaaS DB

    c) Copy file to the management node (please replace here and further sol520.sh with a file of choice)

    ```shell
    scp sol520.sh root@<lic server name>:sol520.sh
    ```

    d) Connect to the management node

    ```shell
    ssh root@<lic server name>
    ```

    e) Make the file executable

    ```shell
    chmod +x sol520.sh
    ```

6. Check the list of DBs and pick a proper one

    ```shell
    dwad_client shortlist
    ```

7. Run the script, entering later the DB password and value for `number_of_days`

    ```shell
    ./sol520.sh [Username] [DB Name]
    ```

8. The last lines of script output should look like

    ```text
    Created zipped archive sol_520_20240526.tar.gz.
    Please transfer it to submit server from support host with
    upload-file.py -f root@cluster:sol_520_20240526.tar.gz -l https://logupload.exasol.com/u/d/<ID>/
    ```

    Disconnect from the management node

    ```shell
    exit
    ```

    and run the `upload-file.py` from script output (filename could be found in the script output)

    ```shell
    upload-file.py -f root@cluster:sol_520_20240526.tar.gz -l https://logupload.exasol.com/u/d/<ID>/
    ```

    The file will be available at `/srv/bugtrack` on `submit01` Support host in Guacamole.

## Additional References

* [KB - DB - How to transfer logs and other files](https://exasol.atlassian.net/wiki/spaces/SUPPORT/pages/6750925)
* [Files Transfer in Exasol](/Environment-Management/files-transfer-in-exasol.md)
* [Connecting to customer DB via Support Host](/Support-and-Services/connecting-to-customer-db-via-support-host.md)

We're happy to get your experiences and feedback on this article!
