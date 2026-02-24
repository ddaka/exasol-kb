---
tool_name: confd_client
doc_type: guide
category: system
title: "Timezone configuration"
summary: "There are at least three time zone settings that affect interaction with Exasol database:"
---
# Timezone configuration

## Overview

There are at least three time zone settings that affect interaction with Exasol database:

1. Server time zone
2. Database time zone
3. Client time zone

Client time zone setting usually applies to JVM used by JDBC-based clients like DBeaver and DbVisualizer and is out of scope for this article.

Server time zone is time zone of COS. For deployments with EXAoperation it is configured on page **Configuration > Network** (see [Configure the Cluster Network](https://docs.exasol.com/db/7.1/administration/on-premise/installation/install_hardware/config_network.htm)).
It's application requires nodes' restart.

In other deployment types it's done by editing the `EXAConf` file.
Please note that even ConfD job [general_settings](https://docs.exasol.com/db/latest/confd/jobs/general_settings.htm) is not capable of changing this setting and it's not planned to improve it in this regard.
As per [SPOT-17185: Always use UTC as the timezone for COS containers](https://exasol.atlassian.net/browse/SPOT-17185) tends to another solution approach.

Server time zone is transferred on the first startup to `TIME_ZONE` session / system parameter (see [ALTER SESSION](https://docs.exasol.com/db/latest/sql/alter_session.htm) / [ALTER SYSTEM](https://docs.exasol.com/db/latest/sql/alter_system.htm)) of the database and becomes Database time zone.

Server time zone controls displaying of `TIMESTAMP` data type. It could be shown via [DBTIMEZONE](https://docs.exasol.com/db/latest/sql_references/functions/alphabeticallistfunctions/dbtimezone.htm) function.

Session / system value controls display format of data type `TIMESTAMP(...) WITH LOCAL TIME ZONE`: [Other considerations for data type TIMESTAMP(3) WITH LOCAL TIME ZONE](https://docs.exasol.com/db/latest/sql_references/data_types/datatypedetails.htm#DateTime).
It could be shown via [SESSIONTIMEZONE](https://docs.exasol.com/db/latest/sql_references/functions/alphabeticallistfunctions/sessiontimezone.htm) function and changed via `ALTER SESSION / ALTER SYSTEM` commands (no need for a DB restart).

The effective Server time zone could be checked inside container via

```shell
cosexec -artw date
```

and also via SQL query

```sql
SELECT dbtimezone;
```

Below are the operational steps to change server timezone.

## Change Server time zone for On-prem

* Edit `EXAConf` and commit changes (inside container).
* Stop the database (inside container)

    ```shell
    confd_client db_stop db_name: <DB_NAME>
    ```

* Stop storage service (inside container)

    ```shell
    csctrl -d
    ```

* Stop c4 services (on host, adapt for rootless accordingly)

    ```shell
    sudo systemctl stop c4_cloud_command
    sudo systemctl stop c4
    ```

* Start c4 services (on host, adapt for rootless accordingly)

    ```shell
    sudo systemctl start c4
    sudo systemctl start c4_cloud_command
    ```

* Start the database (inside container)

    ```shell
    confd_client db_start db_name: <DB_NAME>
    ```

## Change Server time zone for SaaS

* Edit `EXAConf` and commit changes (inside container).
* Stop the database (inside container on master node) or ask customer to stop DB via SaaS UI

    ```shell
    confd_client saas_db_stop account_uuid: <ACCOUNT_UUID> db_uuid: <DB_UUID>
    ```

* Refresh configuration (inside container on n10). Sometimes changes don't apply without it.

    ```shell
    confd_client db_configure db_name: <DB_NAME>
    ```

* Stop n10 instance via AWS Console.
* Start n10 instance via AWS Console.
* Start the database (inside container on master node) or ask customer to start DB via SaaS UI

    ```shell
    confd_client saas_db_start account_uuid: <ACCOUNT_UUID> db_uuid: <DB_UUID>
    ```

## Notes

- In modern setups, UTC is the preferred default unless a business requirement mandates otherwise.
- Always validate timestamp behavior in a test query after restart/config update.

## Additional References

* [SESSIONTIMEZONE](https://docs.exasol.com/db/latest/sql_references/functions/alphabeticallistfunctions/sessiontimezone.htm) / [DBTIMEZONE](https://docs.exasol.com/db/latest/sql_references/functions/alphabeticallistfunctions/dbtimezone.htm) functions in the docs
* [ALTER SESSION](https://docs.exasol.com/db/latest/sql/alter_session.htm) / [ALTER SYSTEM](https://docs.exasol.com/db/latest/sql/alter_system.htm) commands in the docs
* Changing time zone via EXAoperation: [Configure the Cluster Network](https://docs.exasol.com/db/7.1/administration/on-premise/installation/install_hardware/config_network.htm)
* R&D plans regarding this topic: [SPOT-17185: Always use UTC as the timezone for COS containers](https://exasol.atlassian.net/browse/SPOT-17185)
