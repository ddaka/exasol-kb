---
tool_name: confd_client
doc_type: concept
category: architecture
technical_entities:
  - private network
  - public network
  - IP addresses
  - VLAN
  - DHCP
  - subnet
summary: >
  Network infrastructure overview for Exasol deployments — private network
  requirements (consecutive static IPv4, separated VLAN, no traffic filtering)
  and optional public network configuration for external access.
---

# Network

This article provides an overview of the network infrastructure in an Exasol deployment.

The nodes in an Exasol deployment are normally configured with a private network. You can additionally configure a public network on a separate interface to allow direct access to the deployment from outside of the private network.

Exasol does not include any tools for network management. The physical networks must be configured and managed through the network management interfaces on the Linux hosts. You must then specify the private and public IP addresses that will be used by the nodes in the deployment configuration.

---

## Private Network

The nodes in a cluster communicate over the private network. The private network is used to boot and configure nodes, to constantly exchange vitality and configuration information, and to synchronize the database payload.

The nodes must be assigned consecutive and evenly spaced static IPv4 addresses in the same subnet. DHCP can be used if each node always receives the same IP address.

Each private network must be fully separated from other networks. No traffic must pass in or out, and only the dedicated interfaces of the cluster nodes must be wired to this network. A cluster must never exchange traffic with private networks of other clusters.

The nodes must be directly connected to the layer 2 network (VLAN) and traffic must not be filtered.

---

## Public Network

You can additionally configure public IP addresses for the nodes in order to allow clients outside of the private network to connect directly to the database instances and the administration interfaces without being routed through the private network. A public network is optional and not required for installation.

The public IP addresses for the nodes should be consecutive and evenly spaced static IPv4 addresses in the same subnet. DHCP can be used if each node always receives the same IP address.
