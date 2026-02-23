---
tool_name: confd_client
doc_type: guide
category: network
technical_entities:
  - VPN
  - IPSec
  - IKEv1
  - IKEv2
  - site-to-site tunnel
  - support access
  - AES 256
  - SHA-512
  - DH Group 21
summary: >
  VPN access configuration for Exasol support — site-to-site IPSec tunnel
  setup with Phase 1/Phase 2 parameters, required ports/services, and
  supported protocols (IKEv1 and IKEv2).
---

# VPN Access for Support

This article explains how to enable Exasol support access to your system using VPN.

---

## Configuration

To enable support access over VPN, you must configure a site-to-site VPN tunnel. This is an IPSec tunnel with two phases:

- **Phase 1**: AES 256 / SHA-512 / DH Group 21: MODP 2048; Lifetime: 86400 seconds
- **Phase 2**: AES 256 / SHA-512 / PFS Group 21: MODP 2048; Lifetime: 3600 seconds

Exasol supports both the IKEv1 and IKEv2 protocols.

---

## Required Ports and Services

To enable all maintenance tasks, make the following services/ports accessible on the public IP addresses of the cluster and the LOM (if applicable):

| Protocol | Port | Service |
|----------|------|---------|
| TCP | 8563 | Database access |
| TCP | 20002 | SSH access to cluster nodes |
| TCP | 443 | HTTPS access to Administration API and LOM web interface |
| ICMP | — | ECHO REQUEST/REPLY (ping) |
| UDP | 123 | NTP |
