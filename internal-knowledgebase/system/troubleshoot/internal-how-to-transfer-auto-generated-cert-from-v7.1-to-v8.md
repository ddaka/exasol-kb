---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "INTERNAL - How to transfer auto-generated certificates from version 7.1 to 8"
summary: "It is known that direct in-place upgrade from versions prior 8 to version 8 is not possible by design."
---
# INTERNAL - How to transfer auto-generated certificates from version 7.1 to 8

## Overview

It is known that direct in-place upgrade from versions prior 8 to version 8 is not possible by design.

Nevertheless, even in case of backup-restore scenario to new or existing machines customers are willing to change as little as possible in their client applications.

It applies in particular to the connection string.

Imagine upgrade process retains the old public IP addresses and/or DNS names that are used to connect to the database.

If previously customer hasn't uploaded new certificates that are trusted by client applications, those applications might be using a fingerprint.

Fingerprint kind of uniquely identifies the server certificate.
So for a newly installed v8 DB it will be generated anew and will be different.
A solution might have been simple: copy certificates from old version and upload them to the new v8 DB via ConfD job [cert_update](https://docs.exasol.com/db/latest/confd/jobs/cert_update.htm).
However, before version 8 single-file certificate chains where supported, while version 8 requires at least 2 certificates in the chain (see [CHANGELOG: ConfD job cert_update requires a certificate chain](https://exasol.my.site.com/s/article/Changelog-content-21276?language=en_US)).
So if you try to upload those to a version 8 DB, error will be returned:

```text
# confd_client cert_update cert: '"{< ./server_crt.pem}"' key: '"{< ./server_key.pem}"'
JobError: Only one cert in the give cert.
```

If you get this error while uploading 7.1 certificate and key via `confd_client cert_update`, you have to hack the system :)

Namely, manually copy the files to all the expected locations, as it's only a limitation on ConfD job `cert_update` level, DB can work with the old single-file certificate chain.

Important to note:

* If the task can be performed via ConfD job `cert_update` it should be done via this ConfD job, avoiding any manual manipulations.
* All manipulations should be done on all data nodes. Doing on one node and performing `cos_sync_files` on respective files is sufficient.

## How/where to copy the files

Of course, as usual, please take care of backing up files and preserving file permissions that you see in your cluster.

Certificate chain and key need to be copied to the following locations:

### The central folder holding the TLS files

```text
# ls -l /exa/etc/ssl/
total 16
-rw-r--r-- 1 root root 473 Jan  9  2024 ssl.ca
-rw------- 1 root root 241 Jan  9  2024 ssl.ca.key
-rw-r--r-- 1 root root 449 Jan  9  2024 ssl.crt
-rw------- 1 root root 227 Jan  9  2024 ssl.key
```

So files need to go to:

* `server_crt.pem` -> `/exa/etc/ssl/ssl.crt`
* `server_key.pem` -> `/exa/etc/ssl/ssl.key`

If it is allowed to restart the `c4_cloud_command` service (DB downtime is implied), the respective DB Init process will take care of disseminating TLS files to further locations, so nothing else is needed.
Please make sure you are aware about prerequisites for gracefully stopping the `c4_cloud_command` service (like stopping the DB or storage beforehand, depending on versions of c4 and other components).

### The folder holding certificates for all the databases in the cluster

I have only one, called "Exasol", but as you know there might be more. You need to copy the files to all databases that are in use.

```text
# ls -l /exa/etc/dwad/
total 16
-rw-r--r-- 1 exadefusr exausers 1732 Aug  9 17:14 db_Exasol.conf
-rw-r--r-- 1 exadefusr exausers  473 Jan  9  2024 db_Exasol_ca.pem
-rw-r--r-- 1 exadefusr exausers  922 Jan  9  2024 db_Exasol_cert.pem
-rw------- 1 exadefusr exausers  227 Jan  9  2024 db_Exasol_key.pem
```

So files need to go to:

* `server_crt.pem` -> `/exa/etc/dwad/db_[DB_NAME]_cert.pem`
* `server_key.pem` -> `/exa/etc/dwad/db_[DB_NAME]_key.pem`

In case

1. DB version is 8.
2. `-useLegacyTlsMethod=1` DB parameter is not set.
3. Default values of DB parameters responsible for certificate stuff (`-tlsCertificatePath` etc.) aren't manually changed.

DB will be using new TLS files for new connections as soon as files are copied to the target location. If any of the prerequisites is not met, a DB restart is necessary.

### BucketFS

Each BucketFS service has a copy of the server key file. Those can be found via a command like

```text
# find /exa/data/bucketfs/ -name .ssl.key
/exa/data/bucketfs/bfsdefault/.ssl.key
```

So files need to go to:

* `server_key.pem` -> `/exa/data/bucketfs/[BucketFS Service name]/.ssl.key`

for all BucketFS Services.

In my test on 8.29.1 the respective BucketFS service was working as soon as I copied the file - can be tested by

```text
curl -k -v https://localhost:[BucketFS Service port]/
```

In earlier versions I had to kill the respective partition (for them to be started automatically):

```text
[root@n17 ~]# coskillall bucketfsd-bucketfs1
Send signal 15 to partition 103.
```

## Appendix: How to find TLS files for a 7.1 deployment

TLS certificate and key paths used by database are defined by non-public DB parameters `-tlsCertificatePath` and `-tlsPrivateKeyPath`.
Their values could be found by

```text
dwad_client print-setup <db name> | grep -Po 'tlsCertificatePath=\K[^ ]+'

dwad_client print-setup <db name> | grep -Po 'tlsPrivateKeyPath=\K[^ ]+'
```

Usually (== by default) they reside in `/usr/opt/EXASuite-7/EXAClusterOS-<DB version>/var/exaoperation/inst/etc/`.

## Additional References

* [CHANGELOG: TLS for all Exasol drivers](https://exasol.my.site.com/s/article/Changelog-content-6507?language=en_US)
* [Upload TLS Certificate](https://docs.exasol.com/db/latest/administration/on-premise/access_management/tls_certificate.htm)
* ConfD job [cert_update](https://docs.exasol.com/db/latest/confd/jobs/cert_update.htm)
* [CHANGELOG: ConfD job cert_update requires a certificate chain](https://exasol.my.site.com/s/article/Changelog-content-21276?language=en_US)


