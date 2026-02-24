---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to add and trust additional certificates to Exasol (NGA)"
summary: "Sometimes it is necessary to add custom certificates or trust specific hosts to allow and trust secure communication e.g. LDAPs in such scenarios additional certificates need to..."
---
# How to add and trust additional certificates to Exasol (NGA)

## Overview

Sometimes it is necessary to add custom certificates or trust specific hosts to allow and trust secure communication e.g. LDAPs in such scenarios additional certificates need to added to the Exasol system.

This examples shows how to trust **google.com**. All Shell commands are executed inside COS, repeat all Shell commands for all cluster nodes!

## Download the Root PEM

We will use Firefox (version 94.0.2) to download the root pem therefore browse to <https://www.google.com>.

1. Click on the lock symbol in the URL

2. Click on "Connection secure"

3. Click on "More information" this will open a new window

4. Click the tab "security"

5. Click  on "View Certificate" this will open a new tab in Firefox

6. Go to the root certificate "GTS Root R1"

7. Scroll down to the "Download Section"

8. Download either the root pem only or the whole cert chain "Save as PEM file"

## Validating Certs

Without the necessary certs a openssl request will look like shown below. Ensure to be inside **COS** (ssh into COS, check with **cosps**)!

Error message when a host or URL are not trusted:

```bash
[root@n11 ~]# openssl s_client -verify_return_error -connect google.com:443
CONNECTED(00000003)
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify error:num=20:unable to get local issuer certificate
140510851228928:error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1913:
---
no peer certificate available
---
No client certificate CA names sent
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 6641 bytes and written 415 bytes
Verification error: unable to get local issuer certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 20 (unable to get local issuer certificate)
---
```
## Import a PEM

Copy the PEM file to all nodes or directly download them. This example will use **scp**.

```bash
user@host: scp google-chain.pem user@n11:
user@host: ssh n11
[root@n11 ~]# ls *.pem
google-chain.pem
```
Next we add the cert to the trusted keystore of Exasol. Adopt the version as needed!

```bash
[root@n11 ~]# cp google-chain.pem /usr/opt/EXASuite-7/EXARuntime-7.1.0/ssl/certs/$(openssl x509 -hash -noout -in google-chain.pem).0
```
 A new file has been created ending with **.0** (names can be different depending on what has been used as ending from the previous command)

```bash
[root@n11 ~]# ls -lat /usr/opt/EXASuite-7/EXARuntime-7.1.0/ssl/certs/
total 240
drwxr-xr-x 2 root root   4096 Dec  3 10:06 .
-rw-r--r-- 1 root root   8719 Dec  3 10:06 1001acf7.0
drwxr-xr-x 5 root root   4096 Jul 15 13:00 ..
-rw-r--r-- 1 root root 221418 Jul 15 12:56 ca-bundle.crt
```
## Validate the certificate

Inside COS run again the openssl command - now you will see the server certificate and a successful/valid handshake.

```bash
[root@n11 ~]# openssl s_client -verify_return_error -connect google.com:443
CONNECTED(00000003)
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = *.google.com
verify return:1
---
Certificate chain
 0 s:CN = *.google.com
   i:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
 1 s:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
   i:C = US, O = Google Trust Services LLC, CN = GTS Root R1
 2 s:C = US, O = Google Trust Services LLC, CN = GTS Root R1
   i:C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
---
Server certificate
-----BEGIN CERTIFICATE-----
TRUNCATED DATA
-----END CERTIFICATE-----
subject=CN = *.google.com

issuer=C = US, O = Google Trust Services LLC, CN = GTS CA 1C3

---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 6641 bytes and written 488 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
```
