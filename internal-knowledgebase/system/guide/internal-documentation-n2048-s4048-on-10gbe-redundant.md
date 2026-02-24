---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal: N2048 - S4048-ON 10GbE redundant interconnections"
summary: "Configure VLAN-tagged redundant 10GbE interconnects between N2048/N2024 and S4048-ON switches using single-link or port-channel topology."
---
# Internal: N2048 - S4048-ON 10GbE redundant interconnections

## Purpose

Configure redundant 10GbE interconnects between access and aggregation switches for Exasol network VLAN transport.

## VLAN assumptions

Examples below use VLANs `5`, `10`, and `20` with tagged forwarding.

## Scenario A: single interconnect link

On N2048/N2024 (example interface `Te1/0/1`):

```text
en
configure
interface tengigabetethernet 1/0/1
switchport general acceptable-frame-type tagged-only
switchport general allowed vlan add 5,10,20 tagged
no shutdown
exit
exit
wr
```

On S4048-ON (example interface `Te1/48`):

```text
configure
interface tengigabitethernet 1/48
switchport
mtu 9216
no shutdown

interface vlan 5
tagged tengigabitethernet 1/48
exit

interface vlan 10
tagged tengigabitethernet 1/48
exit

interface vlan 20
tagged tengigabitethernet 1/48
exit

show config
```

## Scenario B: two or more interconnect links (port-channel)

On S4048-ON create port-channel and add member interfaces:

```text
configure
interface port-channel 1
channel-member tengigabitethernet 1/47
channel-member tengigabitethernet 1/48
no shutdown
mtu 9216

interface vlan 5
tagged port-channel 1
exit

interface vlan 10
tagged port-channel 1
exit

interface vlan 20
tagged port-channel 1
exit

show interfaces port-channel 1
wr
exit
```

On N2048/N2024 add corresponding interfaces to channel-group:

```text
en
configure
interface tengigabethernet 1/0/1
no channel-group
channel-group 1 mode on
switchport general acceptable-frame-type tagged-only
switchport general allowed vlan add 5,10,20 tagged
no shutdown
exit
exit
wr
```

Repeat for additional member ports.

## Validation

- Port-channel status is `up/up`.
- All member links are `U` (up) on S4048.
- VLAN tagging is present on interconnect link(s).


