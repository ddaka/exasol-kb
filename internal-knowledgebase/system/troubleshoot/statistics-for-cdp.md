---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Statistics for CDP"
summary: "Historically we collected some of statistics from system tables of customer databases and loaded them into \"CDP\" Exasol database."
---
# Statistics for CDP

## Overview

Historically we collected some of statistics from system tables of customer databases and loaded them into "CDP" Exasol database.

There is a PowerBI report called Database Healthcheck ([link](https://app.powerbi.com/groups/me/reports/4aea3d91-0614-4631-add6-9fe6c301f859)) that is based on this data. This report is used by Exasol employees (Solution Engineers, SDM etc.) for sizing and other purposes.

There are different ways to pull the statistics file:

* User can interactively do it via EXAoperation, see [Download Database Statistics](https://docs.exasol.com/db/7.1/administration/on-premise/support/download_stats.htm).
* If deployment has EXAoperation, it's also possible to pull the data programmatically using XMLRPC API: [database.getDatabaseStatistics()](https://github.com/exasol/exaoperation-xmlrpc/blob/master/tools-and-examples/xmlrpc-help/example.md#databasegetdatabasestatistics).
* If deployment doesn't have EXAoperation (pre-NGA, NGA, SaaS, Docker), then some more manual effort is needed. Please see section [Step 1. Get a zip file with statistics](#step-1-get-a-zip-file-with-statistics) for more details.

As of 2025-04-08, there is an ongoing effort to migrate data upload and healthcheck report to the new monitoring stack, to be shown in Grafana.
General process is the following:

1. Get a zip file with statistics
2. Upload the zip file with statistics to CDP
3. Check statistics file upload status
4. Map statistics to a customer for Reporting / PowerBI - **!!!** This is ONLY required for customers where we retrieve statistics for the first time **!!!**
5. Inform the Exasol employee that requested statistics upload that it's done and data should be available in the report in a couple of hours (latest on the next day).

Steps 1.-4. are explained in the dedicated sections below.

## Step 1. Get a zip file with statistics

a. If deployment has EXAoperation and we have access to it, then we pull the file according to [Download Database Statistics](https://docs.exasol.com/db/7.1/administration/on-premise/support/download_stats.htm).

b. If deployment has EXAoperation, but we don't have access to it, we ask customer to follow [Download Database Statistics](https://docs.exasol.com/db/7.1/administration/on-premise/support/download_stats.htm) and upload the resulting file to us.

c. If deployment has no EXAoperation (pre-NGA, NGA, SaaS, Docker) and we have access:

Find file `spool_statistics_complete.sql` in COS: it's always present on cluster nodes (all DB versions) and could also be found [here](https://github.com/exasol/internal-knowledgebase/blob/main/Support-and-Services/attachments/spool_statistics_complete.sql). On cluster nodes for v8 file is typically under `/opt/exasol/db-YOUR_DB_VERSION/exaoperation/statistics/spool_statistics_complete.sql`.

It needs to be called using `exaplus` because of the defines - with some editing, you can run it with any client/driver that supports LOCAL CSV exports.
Call it like this:

```shell
exaplus <connection details> -f spool_statistics_complete.sql -- "<customer name>" "<database name>" "2023-01-01" "2023-10-27"
```

Note: the database user will need `CREATE_SESSION`, `SELECT_ANY_DICTIONARY` and `EXPORT` privileges.

This will dump a few CSV files to the local directory.
You need to put all those files into some folder (best named like `Customer.Database.date1.date2`).
Then you put THE FOLDER into a ZIP file with the same name (e.g. `Customer.Database.date1.date2.zip`).

About (Customer, Database):

* Can be any sensible string - we have to map it manually anyway.
* Should always be the same string for a given system - then we only have to map it once.
* Do not have to match between exaplus parameters and file/folder name - but does not hurt.
* Must (!!!!) be unique to this system - otherwise we can not distinguish them in CDP.

d. If deployment has no EXAoperation (pre-NGA, NGA, Docker), and we don't have access.

Send customer the file [spool_statistics_complete.sql](https://github.com/exasol/internal-knowledgebase/blob/main/Support-and-Services/attachments/spool_statistics_complete.sql) and the exact `exaplus` command that they should run with it (with all placeholders already replaced).
Customer should upload to us the 8 resulting files

* dbsize.csv
* dbusage.csv
* monitor.csv
* profiling.csv
* sqls.csv
* stats_version.csv
* systemevents.csv
* table_stats.csv

Support takes this files and creates a ZIP file according to c. above.

## Step 2. Upload the zip file with statistics to CDP

File upload will be performed using `scp` command.

1. Make sure your public key is added to this IP Address: 192.168.235.2. (DNS: `cdp-staging.db.exasol.com`) - If it is not, open an ITS ticket to do so, mentioning [/wiki/spaces/SD/pages/8816147](https://exasol.atlassian.net/wiki/spaces/SD/pages/8816147).
2. Run the below command in your command line. (Make sure you 'stand' at the same path (directory) where the file to be uploaded is available)

    ```shell
    scp <The ZIP file you want to transfer (without the brackets)> mtransfer@cdp-staging.db.exasol.com:upload/
    ```

    `scp` is the only working command, SFTP (gui tools) is not enabled/supported.

	It's better to not upload files around 06,21,36,51 minutes of each hour, as the respective ETL process might return cryptic error messages if it hits a partially uploaded file.

Network access between server `submit01` and `cdp-staging.db.exasol.com` is open, so it is possible to `scp` files directly from `/srv/bugtrack` (the location where customers upload files via URL given by Salesforce) to `cdp-staging.db.exasol.com`.

## Step 3. Check statistics file upload status

Wait until the statistics are uploaded into CDP (import is triggered every hour at 06,21,36,51 minutes). You can check by running a simple query like:

```sql
select
  *
from
  customer_stats_2016.v_systemevents
where
  1=1
  -- and CUSTOMER_NAME like '%Formue%'
order by
  MEASURE_TIME desc
limit 100
;
```

or checking similar information in PowerBI report [Database Healthcheck](https://app.powerbi.com/groups/me/reports/4aea3d91-0614-4631-add6-9fe6c301f859). If you don't have access to table data in CDP - ask Gerardo Abihaggle to grant it.

If data is not present in the report, one might need to check what went wrong.

Typically, checking the status of the last file uploads is helpful:

```sql
select
	*
from
	CUSTOMER_STATS_2016.H_INPUT_FILE f
where
	1=1
	--and f.FILE_NAME like '%CEWE%'
order by
	f.LOAD_TIME desc
limit 100
;
```

In doubt - ask Aleksandr Lipatev / Stefan Reich / Gerardo Abihaggle.

## Step 4. Map statistics to a customer for Reporting / PowerBI

**!!!This is ONLY required for customers where we retrieve statistics for the first time!!!**

Please ask Gerardo Abihaggle to the mapping (OOO deputy: Stefan Reich).

Task is done when you (or the person adding mapping) sees data in the PowerBI report.

Note: mapping is instantaneous, but PowerBI does not use live connections, but data extracts refreshed every 12 hours (twice a day). However, with the proper rights can be forced (if urgent).

## Limitations

### Worker clusters

* CDP and PowerBI are not built with v8 Main/Worker cluster concept in mind. Therefore:
  * SUM measures in the report could be considered correct when one or more worker clusters are present.
  * MAX measures can sometimes be correct.
  * AVG measures typically don't make sense and shouldn't be used.
* EXAoperation CDP data export takes customer name from license. Some deployment types (e.g. AWS PAYG) have a generic license and DB name. Therefore, file name and content might have "EXASOL" as customer name and "exa_db" as database name,
both making customer identification in CDP hard/impossible. Therefore, if you notice this kind of issue, consider editing the CSV files and replacing the first two columns with more reasonable values. The following command might be helpful

    ```shell
    sed -i -e 's/^EXASOL,exa_db/"<customer name>","<database name>"/' *.csv
    ```

## File format

* Each incoming statistics package (time period for one database) needs to be packed into a **ZIP** file.
* The **name** of the zip-file must match the regular expression `(.+)[-\.](.+)[_\.]([-0-9]+)[_\.]([-0-9]+)\.zip` ("STR-STR_YYYY-MM-DD_YYYY-MM-DD.zip", where the field separators may also be periods instead of mins/underscore.

the actual strings and dates are not important, but for consistency they should be CUSTOMER, DATABASE, FIRST_DAY, LAST_DAY
* Inside the zip file, there **must** be a **folder**. The name of the folder does not matter.
* Within the folder, there are the actual csv files
* The first two columns of every csv file are &lt;License name&gt; and &lt;Database Name&gt; – While they should all be the same, only those in `stats_version.csv` will be evaluated and used to map the package against known databases.

**Example:**

```
Archive:  /opt/cdp/customer_data/20230113-230001/EXASOL-c0001-license-CDP_PROD1_2023-01-05_2023-01-13.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/
     1768  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/table_stats.csv
       59  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/stats_version.csv
     1528  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/systemevents.csv
   484494  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/sqls.csv
  8285163  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/profiling.csv
    25441  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/monitor.csv
    14009  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/dbusage.csv
    26301  2023-01-13 19:45   EXASOL-CDP_PROD1_2023-01-05_2023-01-13/dbsize.csv
---------                     -------
  8838763                     9 files
```

## Additional References

* [PowerBI report Database Healthcheck](https://app.powerbi.com/groups/me/reports/4aea3d91-0614-4631-add6-9fe6c301f859)
* [Download Database Statistics](https://docs.exasol.com/db/7.1/administration/on-premise/support/download_stats.htm)
* [database.getDatabaseStatistics()](https://github.com/exasol/exaoperation-xmlrpc/blob/master/tools-and-examples/xmlrpc-help/example.md#databasegetdatabasestatistics)
* [spool_statistics_complete.sql](https://github.com/exasol/internal-knowledgebase/blob/main/Support-and-Services/attachments/spool_statistics_complete.sql)
