---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to persist OS customizations inside COS container"
summary: "There are situations where we want to make persistent changes inside COS. For instance:"
---
# How to persist OS customizations inside COS container

## Overview

There are situations where we want to make persistent changes inside COS.
For instance:

* Add certificates to OS truststore.
* Add certificates to Script Language Container truststore.
* Create a symlink to a BucketFS file, to mimic `/buckets/...` paths for IMPORT/EXPORT.

Example use cases could be found in [INTERNAL - How to overcome SSL / TLS errors thrown by SELECT / IMPORT / EXPORT](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/internal-how-to-overcome-ssl-tls-errors-thrown-by-select-import-export.md).

The abovementioned OS changes are relatively straightforward.
The problem is that some valid DB manipulations, say update to some DB versions, can recreate COS container completely.
One of the reasons - to update base OS version of the COS container.
This kind of change will erase all changes besides `/exa/` folder inside COS container and reconfigure OS from scratch according to settings found in `EXAConf`.

Therefore, one needs to be prepared to repeat the OS re-customization after each DB update.
It's definitely inconvenient and error-prone.

As a result, we would like to be able to make this kind of customizations persistent.

## Prerequisites

The article applies to Exasol version 8.

We need access into COS.

## How to make customizations persistent

### Approach

The idea is to use the persistency of `/exa/` folder and also the funtionality of file `/exa/etc/rc.local` that is executed on each node startup.

Instead of making the change directly, we write something in the `/exa/etc/rc.local` file which does something like this:

```shell
if [ ! -f /etc/my_customisation_is_there ] ; then
    ... make my customisation.....
    touch /etc/my_customisation_is_there
fi
```

That will re-apply your changes after every container rebuild and it will also document them so that everyone can see which changes are being made.

**NOTE:** Additional nodes won't get the file `/exa/etc/rc.local` automatically, so one needs to separately configure it for new nodes in case of cluster expansion or, for example, cluster addition (elasticity).

Exasol's own `rc.local` follows the general Linux `rc` utility approach to control the boot process (see [Purpose and typical usage of /etc/rc.local](https://unix.stackexchange.com/questions/49626/purpose-and-typical-usage-of-etc-rc-local) and `man` pages - local or similar to [Manual Pages  — RC](https://nxmnpg.lemoda.net/8/rc.local)). It will be called by shell so it should, in particular, contain a proper shebang at the beginning, like

```shell
#!/bin/sh -e
```

Another comment before your customization like

```shell
# Exasol node customizations that should survive a container rebuild
```

also wont hurt.

In general, description above is sufficient to make the necessary changes. Below we would like to suggest a couple of implementation examples to use as more detailed templates, to make sure necessary related files are managed in a predictable way.

Please also note that at least our monitoring agent is already using the `/exa/etc/rc.local` file to start some services, so be sure not to simply rewrite the file, but intelligently append/merge information to/into it.

### Example: Add certificates to OS Java truststore

Customer SBK wanted their IMPORT FROM JDBC statements to connect to some of third party databases without changing JDBC connection string used by IMPORT command. Therefore, it was decided to add CA certificates that signed target server certificates to OS Java truststore.
It was done by adding the following to `/exa/etc/rc.local`:

```shell
CUSTOMISATION_FLAG_FILE='/etc/customization_db2_cert'

if [ ! -f $CUSTOMISATION_FLAG_FILE ] ; then
    keytool -import -alias 21c-db2-daphne01537-gskv-org -file /exa/certificates/21c-db2-daphne01537-gskv-org.pem -cacerts -storepass changeit -noprompt
    touch $CUSTOMISATION_FLAG_FILE
fi

CUSTOMISATION_FLAG_FILE='/etc/customization_mariadb_cert'

if [ ! -f $CUSTOMISATION_FLAG_FILE ] ; then
    keytool -import -alias maria_ca -file /exa/certificates/maria_ca.der -cacerts -storepass changeit -noprompt
    touch $CUSTOMISATION_FLAG_FILE
fi
```

There are two "if" statements serving similar purpose, so we'll discuss only the first one.

We define variable `CUSTOMISATION_FLAG_FILE`

```shell
CUSTOMISATION_FLAG_FILE='/etc/customization_db2_cert'
```

It would become a name of an empty file that tells system that customisation was already performed and doesn't need to be repeated.
File name (here `customization_db2_cert`) could be arbitrary, we used two parts "customization" and "db2_cert" for this change to be a bit self-documented.
A crucial part - it has to reside outside of `/exa/`.

In

```shell
if [ ! -f $CUSTOMISATION_FLAG_FILE ] ; then
```

we check if such a file exists. If not, we execute the body of the customisation

```shell
keytool -import -alias maria_ca -file /exa/certificates/maria_ca.der -cacerts -storepass changeit -noprompt
```

and create the respective file

```shell
touch $CUSTOMISATION_FLAG_FILE
```

so that the next node startup doesn't attempt to execute it again.

Please also note that the certificate to be added to truststore was added to folder `/exa/certificates/` upfront. Key reason - `/exa/` is not touched when COS container is rebuilt. "certificates" subfolder isn't a hard requirement,
but it will still be nice to use unified approach each time we implement such functionality.

Overall, the body of customisation needs to be carefully thought through - where to take file, in which format, which alias to give to it and so on. How to do it is out of scope for this article.

### Example: Create a symlink to a BucketFS file, to mimic "/buckets/..." paths for IMPORT/EXPORT

Here we'll highlight the differences to the the "Add certificates to OS truststore" example above.

Customer Sony needed to configure properly encrypted connection to their PostgreSQL database. That implied providing paths to some custom certificate and key files in the JDBC URL.
Moreover, they wanted to use this connection in PostgreSQL Virtual Schema, so paths should work for both IMPORT FROM JDBC and Java UDFs (adapters).

Based on [Virtual Schemas whose definition requires a path to a particular file](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/internal-how-to-overcome-ssl-tls-errors-thrown-by-select-import-export.md#virtual-schemas-whose-definition-requires-a-path-to-a-particular-file)
the following was added to `/exa/etc/rc.local`:

```shell
CUSTOMISATION_FLAG_FILE='/etc/customization_bucketfs_pgsql_cert'

if [ ! -f $CUSTOMISATION_FLAG_FILE ] ; then
    BUCKETFS_PGSQL_DIR='/buckets/bucketfs1/bucket1/postgre_certs/'
    mkdir -p $BUCKETFS_PGSQL_DIR
    ln -s -t $BUCKETFS_PGSQL_DIR /exa/data/bucketfs/bucketfs1/.dest/bucket1/postgre_certs/RootCert_lcrsffp-pgdbopdsr2-pgb01.smearsys.com.pem
    ln -s -t $BUCKETFS_PGSQL_DIR /exa/data/bucketfs/bucketfs1/.dest/bucket1/postgre_certs/cert.pem
    ln -s -t $BUCKETFS_PGSQL_DIR /exa/data/bucketfs/bucketfs1/.dest/bucket1/postgre_certs/key.pk8
    touch $CUSTOMISATION_FLAG_FILE
fi
```

As a prerequisite, three files `RootCert_lcrsffp-pgdbopdsr2-pgb01.smearsys.com.pem`, `cert.pem` and `key.pk8` were uploaded to BucketFS using CURL.

Having those files in place, the script above creates the necessary folder via

```shell
BUCKETFS_PGSQL_DIR='/buckets/bucketfs1/bucket1/postgre_certs/'

mkdir -p $BUCKETFS_PGSQL_DIR
```

Note again the naming with "PGSQL", to make this part of code distinct.

Then we create three symlinks in this folder via `ln` command like

```shell
ln -s -t $BUCKETFS_PGSQL_DIR /exa/data/bucketfs/bucketfs1/.dest/bucket1/postgre_certs/cert.pem
```

## Additional References

* [INTERNAL - How to overcome SSL / TLS errors thrown by SELECT / IMPORT / EXPORT](https://github.com/exasol/internal-knowledgebase/blob/main/Environment-Management/internal-how-to-overcome-ssl-tls-errors-thrown-by-select-import-export.md)
* [Purpose and typical usage of /etc/rc.local](https://unix.stackexchange.com/questions/49626/purpose-and-typical-usage-of-etc-rc-local)
* [Manual Pages  — RC](https://nxmnpg.lemoda.net/8/rc.local)
