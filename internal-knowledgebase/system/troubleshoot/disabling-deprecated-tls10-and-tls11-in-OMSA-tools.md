---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Disabling Deprecated TLS 1.0 and TLS 1.1 in OMSA-Tools"
summary: "This guide explains how to disable deprecated TLS 1.0 and TLS 1.1 within OMSA-Tools. OMSA (OpenManage System Administrator) is used to check hardware issues on Dell nodes. This..."
---
# Disabling Deprecated TLS 1.0 and TLS 1.1 in OMSA-Tools

## Problem

This guide explains how to disable deprecated TLS 1.0 and TLS 1.1 within OMSA-Tools. OMSA (OpenManage System Administrator) is used to check hardware issues on Dell nodes. This will usually show you if there are any hardware issues on the system.

## Procedure

### Port 443

Edit the following file:

```
/opt/dell/srvadmin/etc/openmanage/oma/ini/omprv.ini
```

Change the `supported_ssl_protocols` to:

```
supported_ssl_protocols="TLSv1.2"
```

### Port 1311

Edit the following file:

```
/opt/dell/srvadmin/lib64/openmanage/apache-tomcat/conf/server.xml
```

Change the `sslEnabledProtocols` within the `<Connector>` tag:

```xml
<Connector SSLEnabled="true"
    ciphers="TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDH_RSA_WITH_AES_256_CBC_SHA,TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDH_RSA_WITH_AES_128_CBC_SHA,TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA"
    clientAuth="false" compression="force" keyPass="${key_password}" keystoreFile="conf/keystore.db"
    keystorePass="${keystore_password}" maxPostSize="6291456" maxThreads="20" port="1311"
    protocol="org.apache.coyote.http11.Http11NioProtocol" scheme="https" secure="true" sslEnabledProtocols="TLSv1.2" />
```

### Restart OMSA Process

Restart the OMSA process with the following command:

```sh
systemctl restart dell-omsa
```

### Testing

To verify the configuration, use the following commands:

### Test Port 443

```sh
openssl s_client -connect localhost:443 -tls1_1
```

Expected output:

```
CONNECTED(00000003)
139676603012352:error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure:ssl/record/rec_layer_s3.c:1543:SSL alert number 40
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 7 bytes and written 128 bytes
Verification: OK
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
        Protocol  : TLSv1.1
        Cipher    : 0000
        Session-ID:
        Session-ID-ctx:
        Master-Key:
        PSK identity: None
        PSK identity hint: None
        SRP username: None
        Start Time: 1664529126
        Timeout   : 7200 (sec)
        Verify return code: 0 (ok)
        Extended master secret: no
---
```

### Test Port 1311

```
openssl s_client -connect localhost:1311 -tls1_1
```

Expected output:

```
CONNECTED(00000003)
139661665457408:error:1409442E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:ssl/record/rec_layer_s3.c:1543:SSL alert number 70
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 7 bytes and written 128 bytes
Verification: OK
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
        Protocol  : TLSv1.1
        Cipher    : 0000
        Session-ID:
        Session-ID-ctx:
        Master-Key:
        PSK identity: None
        PSK identity hint: None
        SRP username: None
        Start Time: 1664529131
        Timeout   : 7200 (sec)
        Verify return code: 0 (ok)
        Extended master secret: no
---
```
