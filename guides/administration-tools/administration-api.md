---
tool_name: confd_client
doc_type: reference
category: administration
subcommands:
  - user_passwd
technical_entities:
  - Administration API
  - REST API
  - OpenAPI
  - Swagger UI
  - ConfD
  - Basic authentication
  - curl
  - HTTPS
  - TLS certificate
summary: >
  Exasol Administration API reference — RESTful API built on ConfD, access via
  HTTPS port 4444, Basic authentication with exaadm group users, Swagger UI for
  interactive exploration, and examples using curl and Python.
---

# Administration API

This article describes the Exasol Administration API.

Exasol Administration API is a RESTful API built on top of ConfD. The Administration API allows you to carry out administration tasks using RESTful commands through various programming languages and interfaces using standard libraries.

The Administration API is defined using the OpenAPI specification. You can use this specification to generate your own code and use the API in the programming language of your choice. The specification is available on all cluster nodes at:

```
https://<node_ip>:4444/openapi.json
```

---

## Access the API

The Administration API is available on all nodes in a cluster. The client that is used to send requests to the API endpoints must be able to access the nodes. If you have not uploaded a TLS certificate to the system, you must configure the client to use insecure server connections when interacting with the API.

The option `--insecure` or `-k` tells curl to bypass the TLS certificate check. This option allows you to connect to a HTTPS server that does not have a valid certificate. Only use this option if certificate verification is not possible and you trust the server.

---

## Authentication

The Administration API supports Basic authentication (BA) with a user name and password. When you deploy Exasol, a user "admin" is automatically created with a password specified in the configuration parameter `CCC_PLAY_ADMIN_PASSWORD`.

In an existing deployment, the password can only be changed using the ConfD job `user_passwd`.

You can create additional users that are able to use the Administration API. The users must belong to the "exaadm" group.

When using basic authentication, an authentication token is added to the "Authorization" header in the HTTP request. The token is a base64 encoding of a string in the format `USER_NAME:PASSWORD`.

To generate an authentication token:

```bash
IFS= read -p $'user: \n' USER_NAME && IFS= read -s -p $'password: \n' PASSWORD && echo -n $USER_NAME:$PASSWORD | base64
```

For example, the user `admin` and the password `exasol` will be encoded as the authorization token `YWRtaW46ZXhhc29s`. You can then add this token to each curl request using the `-H` option:

```bash
curl -H "Authorization: Basic YWRtaW46ZXhhc29s" ...
```

> Only users in the exaadm group are able to authenticate in the Administration API.

---

## Swagger UI

Swagger UI is a visual representation of the Administration API that you can use to generate REST calls, which you can then copy and use in a client. You can also use Swagger UI to interact directly with the Administration API by clicking on Authorize and entering authentication credentials for a system user in the "exaadm" group.

Swagger UI is available on all database nodes at:

```
https://<node_ip>:4444/openapi/index.html
```

---

## Examples

The following example shows how to list the databases in a cluster using curl and Python.

For information about the API endpoints, see **API Endpoints**.

For examples of how to use the API, see **API Examples**.
