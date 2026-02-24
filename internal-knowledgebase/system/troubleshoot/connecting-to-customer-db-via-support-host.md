---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Connecting to customer DB via Support Host"
summary: "Sometimes Support needs to connect to a customer database to run some queries."
---
# Connecting to customer DB via Support Host

## Overview

Sometimes Support needs to connect to a customer database to run some queries.

When customers purchase certain services such as Cluster Administration, this often means that a VPN must be configured between the Exasol site and the customer site.
On Exasol site the connectivity is configured from a particular VM called "Support Host". Each customer has its own Support Host and those Support Hosts are accessed by Support employees via Apache Guacamole.
From Support host one can typically connect (`ssh`) into customer cluster(s) or start some GUI applications if needed:

* DbVisualizer to run queries
* Firefox / Chrome to connect to EXAoperation
* etc.

Some straightforward and standartized tasks could be done by simply connecting via `EXAplus` inside COS.
More interactive tasks benefit from a GUI client. Some examples:

* check for transaction conflicts
* find statements / sessions for further analysis
* check permissions / roles
* analyze performance issues
* check statistics

## Prerequisites

1. Access to **Keeper Password Manager**, create an ITS ticket if required.
2. Access to **Apache Guacamole**, create an ITS ticket if required.
3. Physical dongle called "Yubikey", create an ITS ticket if required.

## How to get into Support host

### Keeper Password Manager

URL: [Keeper](https://keepersecurity.eu/vault/#)

Login via Windows (SSO) credentials.

Entries are grouped in folders devoted to particular customers.
To retrieve a password click on masked password or "Copy Password input value" button nearby.
Use copy/paste as usual, if not working that way just switch to plain text and enter them manually where needed.

### Apache Guacamole

* URL: [Apache Guacamole](https://guacamole.support.exasol.com/#/), only available when VPN is on.
* Login - via own Guacamole credentials ("core password").
* User: your abbreviation (e.g. `all`)
* Password: your core password

Click on the required host, e.g. 'c0036' opens a new window for a Xorg session.

* Session: Xorg
* User: your abbreviation (e.g. `all`)
* Password: your core password + physically click on Yubikey

Once the connection is established

### (First Time) Set up DbVisualizer connection

Using DbVisualizer, we have the option to create and save connections. Please follow the below steps when trying to connect to a database FOR THE FIRST TIME.

* Right click anywhere on the screen and choose `DbVisualizer`

* If this is the first time you are using DBVisualizer on the support host, you will need to add our license key to it so that it uses the PRO edition instead of the FREE edition. If you see a window asking for license, please follow these additional steps:

  * End User License Agreement / "Accept"
  * Welcome to DbVisualizer / "Locate License Key"
  * Choose option "License Key String"
  * Enter the content of file from [DBVisualizer](https://exasol.atlassian.net/wiki/spaces/Prod/pages/7176924/DBVisualizer). The license is refreshed once a year by Product Management - they are in contact with DbVisulizer team.
  * Click "Install License Key" and follow further interface directions

## How to configure DB connection in DbVisualizer

There are many ways to configure connections. Sometimes it can be simple, sometimes we need to adapt more settings. We'll start with a simple approach and gradually add hints necessary for some setups.

### Deployments with EXAoperation

Usually we have access to at least license node, and therefore to EXAoperation. As a result, the simplest way of configuring connection is

1. Find the respective license node among the results of `cat /etc/hosts | grep lic`
2. Go to EXAoperation on that machine by opening `https://<name from the previous step>` in web brower from Support Host; Credentials - in Keeper
3. Locate the DB in `Services` -> `EXASolution` -> `EXASolution Instances`
4. Note down the list of IP addresses, fingerprint and DB port
5. Choose "Create Database Connection" / "Exasol" in DbVisualizer
6. Provide information from step 4. in fields "Database Server", "Certificate Fingerprint", "Database Port" respectively
7. DB credentials ("Database Userid" / "Database Password") are to be found in Keeper. Our read-only user is usually (but not always) called `EXA_DEBUG`.
8. Click "Connect"

If we have network access to all nodes, this approach should just work. If not, please read further.

### Deployments with EXAoperation, no access to data nodes

You configured DB connection according to "Deployments with EXAoperation", tried to connect **several times** but always get errors like

```text
java.net.ConnectException: Connection refused
```

or

```text
java.net.SocketTimeoutException: Connect timed out
```

Likely, we don't have network access from Support Host to data nodes. To make sure that's the case, one can use `ping` and/or `telnet`.

If we are in such situation, we'll need to use "tunnel" functionality of DbVisualizer. To do so, during connection creation please additionally

1. Choose option "Use SSH Tunnel".
2. Create new SSH configuration: in Keeper you'll see if COS/root access for the customer is done via password or via key. In case it's done via key, you'll need to create a respective text file in `~/.ssh/` and assign it proper permissions:

    ```shell
    chmod 600 ~/.ssh/<your file name>
    ```

3. Select new SSH configuration for the DB connection being created.

As connectivity will be happening in fact from license node, connection string would need to use names available in COS, like `n12`, `n13` etc. Please note that full connection string like `n11..20` will fail with

```text
Remote host terminated the handshake
```

so you need to pick one (active) node.

Apart from it, when connecting via tunnel, fingerprint could be provided only via "Certificate Fingerprint" (or via Properties / Driver Properties / fingerprint).
Otherwise, if fingerprint is included directly in connection string (field "Database Server"), you'll also get error

```text
Remote host terminated the handshake
```

### Version 8

Among differences between DB version 8 and previous versions the most important for us are the following:

* v8 databases don't have to include an access node
* There is no central place to find the connection string

How to address these differences:

### V8 databases don't have to include an access node

If that's the case for a particular cluster and we have to use a DbVisualizer tunnel, we do it via one of data nodes.

Keeper would include username and password or private key needed for it.

### There is no central place to find the connection string

To collect IP addresses of data nodes, one needs to `ssh` into one of data nodes - get into "host", not into "COS" - and run `c4 ps`.
The public IP addresses of the nodes are shown in the `EXTERNAL_IP` column. Be careful though: if cluster includes an access node, it will also show up in the outpout of `c4 ps`.

Port used by the database could be found in output of ConfD job [db_info](https://docs.exasol.com/db/latest/confd/jobs/db_info.htm).

---

Considering the two v8 aspects above, the procedure of establishing connection is similar to what is described in sections "Deployments with EXAoperation" and "Deployments with EXAoperation, no access to data nodes".

### SaaS

We connect to SaaS databases not from Support Hosts, but from our laptops.

You can find out how to configure connectivity to a SaaS database in [KB - DB - How to connect to a customer SaaS DB on AWS](https://exasol.atlassian.net/wiki/spaces/SUPPORT/pages/6751014/KB+-+DB+-+How+to+connect+to+a+customer+SaaS+DB+on+AWS)
and [How to get log files from Exasol SaaS systems (temporary solution)](/SaaS/how-to-get-log-files-from-exasol-saas-systems-temporary.md).

## Additional References

* [Keeper](https://keepersecurity.eu/vault/#)
* [Apache Guacamole](https://guacamole.support.exasol.com/#/)
* [DBVisualizer](https://exasol.atlassian.net/wiki/spaces/Prod/pages/7176924/DBVisualizer)
* [Exasol Deployment Tool (c4)](https://docs.exasol.com/db/latest/administration/aws/admin_interface/c4.htm)
* [db_info](https://docs.exasol.com/db/latest/confd/jobs/db_info.htm)
* [KB - DB - How to connect to a customer SaaS DB on AWS](https://exasol.atlassian.net/wiki/spaces/SUPPORT/pages/6751014/KB+-+DB+-+How+to+connect+to+a+customer+SaaS+DB+on+AWS)
* [How to get log files from Exasol SaaS systems (temporary solution)](/SaaS/how-to-get-log-files-from-exasol-saas-systems-temporary.md)
