---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Connecting to ConfD with Python3"
summary: "The current XML-RPC will eventually be replaced by ConfD and this article will teach you how to connect and use the XML-RPC API via Python3."
---
# Connecting to ConfD with Python3

## Overview

The current XML-RPC will eventually be replaced by ConfD and this article will teach you how to connect and use the XML-RPC API via Python3.

## Prerequisites

* Python3 installed
* xmlrpc.client library installed
* A running NGA/Docker system

## Procedure

### How to connect to ConfD via Python3

```
[user@machine ~]# python3
>>> import requests, urllib3, ssl
>>> from xmlrpc.client import ServerProxy
>>> username = "root"
>>> password = "testing" *
>>> xmlrpc_port = 443
>>> master_ip = requests.get("https://172.16.0.11:443/master", verify = False).content.decode("utf-8") **
>>> connection_string = "https://{}:{}@{}:{}".format(username, password, master_ip, xmlrpc_port)
>>> sslcontext = ssl._create_unverified_context()
>>> conn = ServerProxy(connection_string, context = sslcontext, allow_none=True
```
* - You will need to set the password for the root user inside the Exasol container/environment by running the following command and rebooting the container/service:

```
[root@n11 ~]# exaconf passwd-user --name root --passwd testing
```
Of course, you can and should replace **testing** with the password of your choice.

** - This command will set **master_ip** to be the IP address of the master node of the cluster. The IP should be replaced with an IP of any node of the cluster.

Once you execute all of the commands above, you can run the command below to list all possible methods:

```
>>> conn.job_list()
['bucket_add', 'bucket_delete', 'bucket_modify', ....... , 'user_list', 'user_modify', 'user_passwd']
```
For a more convenient view of the list:

```
>>> for i in conn.job_list():
... print(i)
...
bucket_add
bucket_delete
bucket_modify
......
......
user_list
user_modify
user_passwd
```
To execute a method:

```
>>> conn.job_exec("node_list")
{'result_jobid': '12.7', 'result_code': 0, 'result_name': 'OK', 'result_desc': 'Success', 'result_output': {'11': {'id': '11', 'name': 'n11', 'private_net': '172.16.0.11/24', 'private_ip': '172.16.0.11', 'uuid': '1676CC713038417896A7683B0D68885EC03FD5C0', 'disks': {'disk1': {'name': 'disk1', 'component': 'exastorage', 'devices': ['/dev/mapper/NGAVG-NGALV'], 'ephemeral': False, 'direct_io': True}}, 'docker_volume': '/exa/etc/n11', 'exposed_ports': [[8563, 8574], [2580, 2591]]}, '12': {'id': '12', 'name': 'n12', 'private_net': '172.16.0.12/24', 'private_ip': '172.16.0.12', 'uuid': 'EB354B9823C841A99C252BA96FE6CD74F28F2C1B', 'disks': {'disk1': {'name': 'disk1', 'component': 'exastorage', 'devices': ['/dev/mapper/NGAVG-NGALV'], 'ephemeral': False, 'direct_io': True}}, 'docker_volume': '/exa/etc/n12', 'exposed_ports': [[8563, 8575], [2580, 2592]]}, '13': {'id': '13', 'name': 'n13', 'private_net': '172.16.0.13/24', 'private_ip': '172.16.0.13', 'uuid': 'A4B6D6AA02A84040A73F90279FF65EF26EE94147', 'disks': {'disk1': {'name': 'disk1', 'component': 'exastorage', 'devices': ['/dev/mapper/NGAVG-NGALV'], 'ephemeral': False, 'direct_io': True}}, 'docker_volume': '/exa/etc/n13', 'exposed_ports': [[8563, 8575], [2580, 2592]]}}}
```
The output is a dictionary, so you can sort through it pretty easily:

```
>>> conn.job_exec("node_list")['result_output']['11']['disks']
{'disk1': {'name': 'disk1', 'component': 'exastorage', 'devices': ['/dev/mapper/NGAVG-NGALV'], 'ephemeral': False, 'direct_io': True}}
```
This is the basics of connecting to NGA/Docker for management via ConfD.

## Examples

### Things to note

Since the outputs are formatted as JSON, it's recommended to import pprint too, if you want the output to be easier to read.

```
>>> import pprint
>>> exaprint = pprint.pprint
```
### . View Database Status

To view the status of a specific database:

```
SYNTAX: conn.job_exec("db_state", {'params': {'db_name': '**<db_name>**'}})

>>> exaprint(conn.job_exec("db_state", {'params': {'db_name': 'DB1'}}))
{'result_code': 0,
'result_desc': 'Success',
'result_jobid': '12.23',
'result_name': 'OK',
'result_output': 'running'}

>>> conn.job_exec("db_state", {'params': {'db_name': 'DB1'}})['result_output']
'running'
```
### . View Database Info

To view some usage info and nodes' info of a database:

```
SYNTAX: conn.job_exec("db_info", {'params': {'db_name': '<db_name>'}})

>>> exaprint(conn.job_exec("db_info", {'params': {'db_name': 'DB1'}}))
{'result_code': 0,
 'result_desc': 'Success',
 'result_jobid': '12.24',
 'result_name': 'OK',
 'result_output': {'connectible': 'Yes',
                   'connection string': '172.16.0.11:8563',
                   'info': '',
                   'name': 'DB1',
                   'nodes': {'active': ['n11', 'n12'],
                             'failed': [],
                             'reserve': ['n13']},
                   'operation': 'None',
                   'persistent volume': 'DataVolume1',
                   'quota': 0,
                   'state': 'running',
                   'temporary volume': 'v0002',
                   'usage persistent': [{'host': 'n11',
                                         'size': '25 GiB',
                                         'used': '6.4297 MiB',
                                         'volume id': 0},
                                        {'host': 'n12',
                                         'size': '25 GiB',
                                         'used': '6.2891 MiB',
                                         'volume id': 0}],
                   'usage temporary': [{'host': 'n11',
                                        'size': '1 GiB',
                                        'used': '0 B',
                                        'volume id': 2},
                                       {'host': 'n12',
                                        'size': '1 GiB',
                                        'used': '0 B',
                                        'volume id': 2}]}}
```
## References

[Using XML-RPC to manage Docker clusters](https://exasol.my.site.com/s/article/Using-XML-RPC-to-manage-Docker-clusters)

[ConfD_XMLRPC](https://exasol.atlassian.net/wiki/spaces/RD/pages/12159574/ConfD+XMLRPC)
