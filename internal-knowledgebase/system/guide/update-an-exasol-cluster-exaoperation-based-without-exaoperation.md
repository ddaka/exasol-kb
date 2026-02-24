---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Update an Exasol cluster (EXAoperation-based) without EXAoperation on AWS"
summary: "Legacy AWS workflow: patch an EXAoperation-based image, create a custom AMI, and deploy cluster from that AMI."
---
# Update an Exasol cluster (EXAoperation-based) without EXAoperation on AWS

## Purpose

When EXAoperation update path is unavailable, pre-patch an Exasol EC2 image and redeploy from a custom AMI.

## Scope note

This is a legacy EXAoperation-based flow. For standard cluster updates, prefer c4 workflows in `documents/c4/c4_updating.md`.

## Prerequisites

- AWS console/CLI access.
- Latest security patch package downloaded.
- SSH key access to launched EC2 instance.

## Procedure

## 1) Launch patching instance

Launch an EC2 instance from the target Exasol AMI with:

* Network
* Subnet
* Auto-assign Public IP (optional if using Elastic IP)
* Empty user data
* Valid SSH key pair
* Security group with SSH access

## 2) Copy patch package to instance

Download appropriate patch package, then upload it:

```bash
scp -i <path_to_key> <patch.pkg> ec2-user@<ip_address>:/home/ec2-user
```
## 3) Unpack patch archive

```bash
ssh -i <path_to_key> ec2-user@<ip_address>
sudo su
cd /home/ec2-user
mkdir -p dir1/dir2
tar xvf <patch.pkg> -C dir1/
tar xvf dir1/<EXAClusterOS_Patchlevel_archive>.tar.gz -C dir1/dir2/
```

## 4) Apply security patch

Copy extracted OS patch tarball into package directory:

```bash
cp ./dir1/dir2/<CentOS_Patchlevel_OS_archive>.tar.gz "$COS_DIRECTORY/var/clients/packages/"
```

Run patch application:

```bash
$COS_DIRECTORY/sbin/apply_os_security_updates
```

## 5) Create custom AMI

Shut down the instance:

```bash
poweroff
```

In AWS console:
- `EC2 -> Instances -> Image and templates -> Create image`
- Track AMI creation in `EC2 -> Images -> AMIs`

## 6) Deploy cluster with patched AMI

Deploy via Cloudtools/CloudFormation and replace default AMI ID with the custom patched AMI ID.

Cloudtools: <https://cloudtools.exasol.com/#/>

## Canonical references

- `documents/c4/c4_updating.md`
- `documents/c4/c4_troubleshooting.md`
