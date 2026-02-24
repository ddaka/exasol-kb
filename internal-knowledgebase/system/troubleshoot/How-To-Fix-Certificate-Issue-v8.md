---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to fix certificate issue in version 8"
summary: "This article applies for version 8."
---
# How to fix certificate issue in version 8

## Problem

This article applies for version 8.

In the past we had issues where a certificate for a database was not working.

## Procedure

You then either need to create a new certificate or copy it from an existing cluster if you want to have the same certificate.

### Create a new certificate

You can create a new certificate with the following script. This will also already sync the script to all nodes.
You need to execute this script inside of COS.
```
#!/bin/bash
CA_LOCATION="/usr/local/share/ca-certificates"
CA_PREFIX_NAME="F*"
for CERT_DIR in $(find / -type d -wholename '*/etc/ssl/certs' -o -wholename '*/runtime*/ssl/certs' 2>/dev/null); do
    for CA in $(ls ${CA_LOCATION}/${CA_PREFIX_NAME}); do
        if [ "$1" = "test" ]; then
            echo "cp "$CA" "$CERT_DIR"/$(openssl x509 -hash -noout -in "$CA").0"
        elif [ "$1" = "install" ]; then
            cp "$CA" "$CERT_DIR"/$(openssl x509 -hash -noout -in "$CA").0
            cos_sync_files "$CERT_DIR"
            psh update-ca-certificates
        else
            echo "Provide argument: 'test' or 'install'"
            exit 1
        fi
    done
done
```
### Copy from existing cluster

The certificates is located in
```
/usr/local/share/ca-certificates/
```
Copy it from an existing cluster to the other cluster into the same folder.
Then sync it to all nodes
```
cos_sync_files /usr/local/share/ca-certificates/
```
Finally use
```
psh update-ca-certificates
```
to update the certificate.
