---
tool_name: confd_client
doc_type: reference
category: network
technical_entities:
  - firewall
  - ports
  - TCP
  - UDP
  - BucketFS
  - Administration API
  - Exasol Admin
  - database client
  - SSH
  - HTTPS
  - NTP
  - DNS
  - LDAP
  - FTP
  - FTPS
  - JDBC
  - Oracle
  - MySQL
  - PostgreSQL
  - DB2
  - SQL Server
  - Sybase
summary: >
  Firewall and port settings reference — required inbound/outbound firewall
  rules and complete tables of default incoming and outgoing TCP/UDP ports for
  Exasol services, database connections, and external system integrations.
---

# Firewall and Port Settings

This article describes the necessary firewall configuration and the default ports used by Exasol.

---

## Firewall Rules

The following inbound/outbound traffic must be allowed for your Exasol deployment:

- SQL client connections to the database
- SSH access to all cluster nodes
- HTTPS access to the Administration API
- NTP
- DNS

Optional:
- LDAP

---

## Default Ports

The following tables describe the default ports used in Exasol for different protocols and services. Many of these protocols and database management systems can be manually configured to use other ports.

### Incoming Ports

| Protocol | Port | Source | Destination | Description |
|----------|------|--------|-------------|-------------|
| TCP | 2580 | Database client | Database nodes | Default BucketFS service. You must assign a HTTP or HTTPS port for each BucketFS service that you create. |
| TCP | 4444 | Customer network | All nodes | HTTPS access to Administration API |
| TCP | 8443 | Customer network | Database nodes | HTTPS access to Exasol Admin server |
| TCP | 8563 | Database client | Database nodes | Exasol database client connection port |
| TCP | 20000–21000 | Database nodes (source) | Database nodes (target) | Data transfer between nodes |
| TCP | 20002 | Customer network | All nodes | Shell access to EXACluster Operating System (COS) on all nodes |
| TCP | 20003 | Customer network | All nodes | XML-RPC access to ConfD |

### Outgoing Ports

| Protocol | Port | Source | Destination | Description |
|----------|------|--------|-------------|-------------|
| TCP | 20 | Database nodes | FTP server | FTP data port for IMPORT/EXPORT |
| TCP | 21 | Database nodes | FTP server | FTP control port for IMPORT/EXPORT |
| TCP | 53 | All nodes | DNS server | DNS port |
| TCP | 80 | Database nodes | HTTP server | HTTP port for IMPORT/EXPORT |
| TCP | 123 | All nodes | NTP server | NTP port |
| TCP | 389 | All nodes | LDAP server | LDAP port |
| TCP | 443 | Database nodes | HTTPS server | HTTPS port for IMPORT/EXPORT |
| TCP | 636 | All nodes | LDAPS server | LDAPS port |
| TCP | 990 | Database nodes | FTPS server | FTPS control port for IMPORT/EXPORT |
| TCP | 1433 | Database nodes | SQL Server database | SQL Server port (JDBC connection) |
| TCP | 1521 | Database nodes | Oracle database | Oracle server port (JDBC/ORA connection) |
| TCP | 3306 | Database nodes | MySQL database | MySQL server port (JDBC connection) |
| TCP | 5000 | Database nodes | Sybase ASE database | Sybase ASE server port (JDBC connection) |
| TCP | 5432 | Database nodes | PostgreSQL database | PostgreSQL server port (JDBC connection) |
| TCP | 8563 | Database nodes | Database client | Exasol database client connection port |
| TCP | 20000–21000 | Database nodes (source) | Database nodes (target) | Data transfer between nodes |
| TCP | 49152–65535 | Database nodes | FTP server | FTP/FTPS PASV mode data ports |
| TCP | 50000 | Database nodes | DB2 database | DB2 server port (JDBC connection) |
