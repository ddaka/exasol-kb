---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "PLAYBOOK: How to create a Java Keystore and Truststore to use with SSL on a JDBC connection"
summary: "Some of the dialects used by Virtual Schemas, can use JDBC connection together with Transport Layer Security (TLS)/Secure Sockets Layer (SSL). However, if the foreign system only..."
---
# PLAYBOOK: How to create a Java Keystore and Truststore to use with SSL on a JDBC connection

## Problem

Some of the dialects used by Virtual Schemas, can use JDBC connection together with Transport Layer Security (TLS)/Secure Sockets Layer (SSL). However, if the foreign system only allows secure connections (enforce security) then a non-secure connection won't be possible and enabling SSL over the JDBC is mandatory.

## Diagnosis

If the foreign system does not allow a non secure connection to be stablished then you will get an error like the example below and therefore the only option is to configure your system to be able to access a certificate.

```sql
[ETL-5] JDBC-Client-Error: Connecting to 'jdbc:hive2://SERVER:PORT;sslTrustStore=<customer_name>.truststore.jks;sslTrustStorePassword=XXXXX;ssl=1' as user='USER' failed: [Cloudera][HiveJDBCDriver](500164) Error initialized or created transport for authentication: javax.net.ssl.SSLException: java.lang.StringIndexOutOfBoundsException: String index out of range: 10. (Session: 1652173049127785741)
```

or

```sql
[42636] ETL-5402: JDBC-Client-Error: Connecting to 'jdbc:hive2://SERVER:PORT;AuthMech=3;AllowSelfSignedCerts=1;CAIssuesCertNamesMismatch=1;SSL=1;SSLKeyStore=/buckets/bfsdefault/bucket1/<customer_name>.keystore.jks;SSLKeyStorePwd=XXXXX;SSLTrustStore=/buckets/bfsdefault/bucket1/<customer_name>.truststore.jks;SSLTrustStorePwd=XXXXX' as user='USER' failed: [Cloudera][HiveJDBCDriver](500164) Error initialized or created transport for authentication: /buckets/bfsdefault/bucket1/<customer_name>.keystore.jks (No such file or directory). (Session: 1672285628399753892)
```

## Explanation

The 3rd party system is enforcing secure connections and therefore enabling SSL over the JDBC connection is mandatory here.

In order to set this up, the Customer should provide the client certificate and CA Authority certificate that should be used together with JDBC.

In this guide, I am going to show how to create the Java Keystore and Truststore that must be use to get a secure connection between Exasol and a 3rd party system like Hive or Oracle. Then they can be uploaded to BucketFS and referenced in the JDBC connection URL.

## Procedure

Please see our suggestions in order to get this issue solved below:

## Separate into Client and CA certificates

Please note this step is optional if the certificates are already separate.

Usually customers provide a self signed certificate for Client and CA in PEM format.

For example:

```
-----BEGIN CERTIFICATE-----
<CLIENT_CERT_REDACTED>
-----END CERTIFICATE-----
# Customer Issuing CA
-----BEGIN CERTIFICATE-----
<CA_CERT_REDACTED>
-----END CERTIFICATE-----
```

Therefore, create two files from this single file.

Copy the first part (including the BEGIN CERTIFICATE and END CERTIFICATE strings) into **client.pem** file:

```
-----BEGIN CERTIFICATE-----
<CLIENT_CERT_REDACTED>
-----END CERTIFICATE-----
```

Similarly, copy the second part into **ca.pem** file:

```
# Customer Issuing CA
-----BEGIN CERTIFICATE-----
<CA_CERT_REDACTED>
-----END CERTIFICATE-----
```

## Create Keystore and Truststore

Now we need to create a keystore that contains the Client certificate and a truststore that contains the CA certificate.

### Create a keystore and import Client and CA certificates

```bash
keytool -noprompt -keystore ./<customer_name>.keystore.jks -alias <customer_name> -import -file client.pem -storepass <KEYSTORE_PASSWORD>
```

Please substitute the <customer_name> (for example, **exasol.keystore.jks**) and <KEYSTORE_PASSWORD> accordingly.

Similarly, import the CA certificate:

```bash
keytool -noprompt -keystore ./<customer_name>.keystore.jks -alias CARoot -import -file ca.pem -storepass <KEYSTORE_PASSWORD>
```

### Create a truststore and import the CA certificate

```bash
keytool -noprompt -keystore ./<customer_name>.truststore.jks -alias CARoot -import -file ca.pem -storepass <TRUSTSTORE_PASSWORD>
```

### Upload files to BucketFS

Upload these two files, <customer_name>.keystore.jks and <customer_name>.truststore.jks to a bucket into BucketFS.

## Imitating Bucket Path for ExaLoader

Since we have uploaded the files to the BucketFS path, the Virtual Schema adapter can reach and access them. However, the ExaLoader will throw an exception that the files do not exist. In order to solve this issue, you can imitate the BucketFS path for ExaLoader.

For example, the files are uploaded to a bucket named bucket1, then run these commands on **all** Exasol datanodes including the "reserve" node:

```bash
mkdir -p /buckets/bfsdefault/<bucket_name>
cd /buckets/bfsdefault/<bucket_name>
ln -s ../../../d02_data/bfsdefault/<bucket_name>/<customer_name>.keystore.jks
ln -s ../../../d02_data/bfsdefault/<bucket_name>/<customer_name>.truststore.jks
```

Please substitute the <bucket_name> (for example, **/buckets/bfsdefault/bucket1**) accordingly.

## JDBC Connection URL

Enable the SSL for the connection URL by adding the properties `SSl`, `SSlKeyStore`, `SSlKeyStorePwd`, `SSlTrusttore` and `SSlTrustStorePwd` similar to the line below:

```
jdbc:hive2://<host>:10000;AuthMech=3;SSL=1;SSLKeyStore=/buckets/bfsdefault/<bucket_name>/<customer_name>.keystore.jks;SSLKeyStorePwd=<KEYSTORE_PASSWORD>;SSLTrustStore=/buckets/bfsdefault/<bucket_name>/<customer_name>.truststore.jks;SSLTrustStorePwd=<TRUSTSTORE_PASSWORD>
```

If the certificates are self signed, then add the `AllowSelfSignedCerts` property as well:

```
jdbc:hive2://<host>:10000;AuthMech=3;SSL=1;SSLKeyStore=/buckets/bfsdefault/<bucket_name>/<customer_name>.keystore.jks;SSLKeyStorePwd=<KEYSTORE_PASSWORD>;SSLTrustStore=/buckets/bfsdefault/<bucket_name>/<customer_name>.truststore.jks;SSLTrustStorePwd=<TRUSTSTORE_PASSWORD>;AllowSelfSignedCerts=1
```

## Additional References

* [Cloudera HIVE JDBC Documentation - Configuring SSL](https://docs.cloudera.com/documentation/other/connectors/hive-jdbc/latest/Cloudera-JDBC-Driver-for-Apache-Hive-Install-Guide.pdf)
* [Oracle Keytool Documentation](https://docs.oracle.com/en/java/javase/11/tools/keytool.html)
* [Differences between PEM, DER, P7B/PKCS#7, PFX/PKCS#12 certificates](https://myonlineusb.wordpress.com/2011/06/19/what-are-the-differences-between-pem-der-p7bpkcs7-pfxpkcs12-certificates/ "Differences")
