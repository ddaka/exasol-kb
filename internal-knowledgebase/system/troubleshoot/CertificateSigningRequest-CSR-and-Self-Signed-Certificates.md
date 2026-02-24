---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Certificate Signing Requests (CSR) and Self-Signed Certificates"
summary: "A **Certificate Signing Request (CSR)** is a file containing information about an organization and its public key, used to apply for a TLS certificate from a Certificate Authority..."
---
# Certificate Signing Requests (CSR) and Self-Signed Certificates

## . What is a Certificate Signing Request (CSR)

A **Certificate Signing Request (CSR)** is a file containing information about an organization and its public key, used to apply for a TLS certificate from a Certificate Authority (CA).

The CSR allows the CA to generate a TLS certificate, which involves the following steps:

1. **Generating a Private Key**: A private key is generated using a key pair generation algorithm, such as RSA.
2. **Creating the CSR**: The CSR is created by providing the necessary information (e.g., contact information, IP addresses, DNS hostnames) and including the public key generated in the previous step.
3. **Submitting the CSR**: The CSR is submitted to the CA, along with any required documentation, to apply for the certificate. Examples of CAs include DigiCert, GoDaddy, and Verisign.

The CA verifies the organization's identity using the information in the CSR and creates a TLS certificate. This certificate is then sent back to the organization.

> **Note**: Keep the private key safe and secure. It should not be shared with the CA.

## . How to Create a CSR

### Prerequisites

Ensure that **OpenSSL** is installed on your system.

> **Disclaimer**: The steps below are tested on a Linux machine. Windows and Mac may have some variations.

### A: Create a Private Key

Run the following command to generate a private key:

```shell
openssl genrsa -out key.pem 2048
```

This command generates a key file in your current directory named `key.pem`.

- The algorithm used to create the key is **RSA**, which is why the command includes the `genrsa` subcommand.
- The key length, specified at the end as `2048`, sets the key size. If omitted, the default value of `512` will be used.

### B. Create a Configuration File and Fill in the Options

In this example, the file is called `san.conf`.

```text
[req]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_req
prompt             = no
[req_distinguished_name]
countryName                = DE
stateOrProvinceName        = N/A
localityName               = Nuremberg
organizationName           = EXASOL AG
commonName                 = *example.com.
[req_ext]
subjectAltName             = @alt_names
[v3_req]
subjectAltName             = @alt_names
[alt_names]
IP.1                       = 10.17.1.80
IP.2                       = 10.17.1.81
IP.3                       = 10.17.1.82
DNS.1                      = node1.example.com
DNS.2                      = node2.example.com
DNS.3                      = node3.example.com
```

This configuration file is necessary for including the IP addresses or domain names of the cluster nodes in the CSR. The TLS certificate generated from this CSR will contain these IP addresses and DNS names in the Subject Alternative Names section.

Note: If DNS names are not required, they can be omitted. It is essential to include the correct IP addresses or domain names in the TLS certificate to prevent security warnings when logging into EXAoperation via a browser or connecting to the database with a DB client.

### C. Use the OpenSSL Command to Generate the CSR

Run the following command to generate the CSR:

```shell
openssl req -new -key key.pem -out server.csr -config san.conf
```

To read the content of the CSR, use the command below:

```shell
openssl req -noout -text -in server.csr
```

## . How to Create a Self-Signed Certificate

The `san.conf` file created in step 2 is also used to create a self-signed certificate.

The self-signed TLS certificate will include the IPs or domain names of the cluster nodes in its **Subject Alternative Names** section.

After creating `san.conf`, use the following command:

```shell
openssl req -x509 -nodes -days 3942 -newkey rsa:2048 -keyout key.pem -out cert.pem -config san.conf
```

The command above will generate a self-signed certificate named cert.pem and a private key named key.pem with a length of 2048 bits.

|req|Subcommand for OpenSSL.|
|-|-|
|-x509|Specifies the x509 standard, which is used to issue the certificate.|
|-nodes|Omits encryption of the certificate with the DES encryption algorithm.|
|-days 3942|Sets the certificate’s expiration to 3942 days (about 11 years).|
|-newkey rsa:2048|Creates a new private RSA key with a length of 2048 bits.|
|-keyout|Specifies the name of the generated private key.|
|-out|Specifies the name of the certificate.|
|-config|Specifies the name of the configuration file.|

## . How to Read the Content of a Certificate

Use the following command to decode the certificate and display all the information it includes:

```shell
openssl x509 -in cert.pem -text -noout
```

The command above will decode the certificate and display its details. Note that if you use cat cert.pem, you will only see encoded text.

## . Where are the TLS Certificate and Private Key Stored in the Cluster?

Run the command below to find the path of the certificate on the cluster:

```shell
dwad_client list | grep -Po 'tlsCertificatePath=\K[^ ]+'
```
