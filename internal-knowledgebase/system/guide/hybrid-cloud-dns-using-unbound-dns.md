---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Hybrid cloud DNS using Unbound DNS"
summary: "Set up Unbound forwarders for hybrid on-prem/cloud name resolution and point Exasol nodes to the new DNS resolvers."
---
# Hybrid cloud DNS using Unbound DNS

## Purpose

Enable DNS resolution between on-premises and cloud environments for Exasol deployments.

## Prerequisites

- On-prem DNS servers.
- VPN connectivity between on-prem and cloud VPC.
- Exasol cluster in target VPC.

## Recommended architecture

- Deploy at least two Unbound DNS instances across different AZs.
- Restrict network access to required subnets and protocols.

## 1) Deploy Unbound instance

Example user-data bootstrap (adjust values):

```bash
#!/bin/bash
vpc_dns=vpc.dns.server.ip
onprem_domain=local.domain.com
onprem_dns=local.dns.server.ip

yum update -y
yum install -y gcc openssl-devel expat-devel
yum install -y unbound

cat << EOF2 | tee /etc/unbound/unbound.conf
server:
        chroot: ""
        logfile: "/var/log/unbound.log"
        verbosity: 1
        log-queries: yes
        interface: 0.0.0.0
        access-control: 0.0.0.0/0 allow
forward-zone:
        name: "."
        forward-addr: ${vpc_dns}
forward-zone:
        name: "${onprem_domain}"
        forward-addr: ${onprem_dns}
EOF2

touch /var/log/unbound.log
chown unbound:unbound /var/log/unbound.log
systemctl start unbound
systemctl enable unbound
```

## 2) Security group rules

Allow inbound:

- UDP 53 from Exasol cluster subnet(s)
- UDP 53 from on-prem subnet(s)
- TCP 22 for administration

## 3) Point Exasol nodes to Unbound resolvers

Set DNS servers in EXAoperation network settings (or equivalent control plane):

- DNS Server 1: Unbound resolver A
- DNS Server 2: Unbound resolver B

Alternative via ConfD (canonical command reference): `general_settings` / `NameServers`.

## Validation

- Confirm `/etc/resolv.conf` on nodes contains Unbound IPs.
- Validate name resolution for cloud and on-prem domains.
- Validate DB/client connectivity relying on cross-domain DNS.

## References

- <https://aws.amazon.com/de/blogs/security/how-to-set-up-dns-resolution-between-on-premises-networks-and-aws-by-using-unbound/>
- `documents/cos/confd-system-and-infrastructure.md` (NameServers settings)


