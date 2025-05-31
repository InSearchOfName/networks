# LAN Configuration Guide

This guide covers essential LAN configuration topics for Cisco switches, including VLANs, trunking, EtherChannel, spanning tree, switch management IP, port security, duplex/speed settings, and verification commands. Each section includes configuration examples and explanations.

---

## VLAN Configuration

**VLANs (Virtual Local Area Networks) logically segment a network into different broadcast domains.**

### Create and Name a VLAN

```
S1(config)# vlan 20
S1(config-vlan)# name management
S1(config-vlan)# end
```
**Explanation:**
- `vlan 20`: Creates VLAN 20.
- `name management`: Names the VLAN "management".

---

### Assign Ports to a VLAN

```
S1(config)# interface range gigabitEthernet 1/0/1-10
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 20
S1(config-if-range)# end
```
**Explanation:**
- `switchport mode access`: Sets the port as an access port.
- `switchport access vlan 20`: Assigns the port(s) to VLAN 20.

---

### Configure a Trunk Port

```
S1(config)# interface gigabitEthernet 1/0/24
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk allowed vlan 10,20,30,40
S1(config-if)# switchport trunk native vlan 99
S1(config-if)# end
```
**Explanation:**
- `switchport mode trunk`: Sets the port as a trunk.
- `switchport trunk allowed vlan ...`: Specifies which VLANs are allowed on the trunk.
- `switchport trunk native vlan 99`: Sets VLAN 99 as the native VLAN.

---

### Port Security

```
S1(config)# interface range gigabitEthernet 1/0/1-24
S1(config-if-range)# switchport port-security
S1(config-if-range)# switchport port-security mac-address sticky
```
**Explanation:**
- `switchport port-security`: Enables port security.
- `switchport port-security mac-address sticky`: Learns and sticks MAC addresses dynamically.

---

## EtherChannel (Link Aggregation)

**EtherChannel bundles multiple physical links into a single logical link for increased bandwidth and redundancy.**

### Configure EtherChannel with LACP

```
S1(config)# interface range GigabitEthernet 1/0/1-2
S1(config-if-range)# channel-group 1 mode active
S1(config-if-range)# exit
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
S1(config-if)# switchport trunk allowed vlan 1,2,20
```
**Explanation:**
- `channel-group 1 mode active`: Adds interfaces to EtherChannel group 1 using LACP.
- `interface port-channel 1`: Configures the logical EtherChannel interface.

---

## Spanning Tree Protocol (STP)

**STP prevents loops in a switched network by blocking redundant paths.**

### Configure Rapid PVST+ and Set Root Bridge

```
spanning-tree vlan 2
spanning-tree mode rapid-pvst
spanning-tree vlan 2 root primary
```
**Explanation:**
- `spanning-tree vlan 2`: Enables STP for VLAN 2.
- `spanning-tree mode rapid-pvst`: Uses Rapid PVST+.
- `spanning-tree vlan 2 root primary`: Makes this switch the root bridge for VLAN 2.

---

## Switch Management IP Address

**Assign an IP address to a VLAN interface for switch management.**

```
S1(config)# interface vlan 99
S1(config-if)# ip address 172.17.99.1 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# end
S1# ip default-gateway 172.17.99.254
```
**Explanation:**
- `interface vlan 99`: Configures the SVI for management.
- `ip address ...`: Assigns the management IP.
- `ip default-gateway ...`: Sets the default gateway for management traffic.

---

## Duplex and Speed Settings

**Set interface duplex and speed for optimal performance.**

```
S1(config)# interface GigabitEthernet 1/0/1
S1(config-if)# duplex full
S1(config-if)# speed 100
```
**Explanation:**
- `duplex full`: Enables full duplex.
- `speed 100`: Sets speed to 100 Mbps.

---

## Useful Show Commands

**Verify and troubleshoot LAN configuration with these commands:**

```
S1# show vlan brief
S1# show interfaces status
S1# show interfaces trunk
S1# show port-security interface [interface]
S1# show etherchannel summary
S1# show interfaces port-channel 1
S1# show spanning-tree
S1# show running-config interface vlan 99
S1# show ip interface brief
S1# show interfaces [interface]
S1# show interfaces description
```
**Explanation:**
- `show vlan brief`: Lists VLANs and port assignments.
- `show interfaces status`: Shows interface status, speed, and duplex.
- `show interfaces trunk`: Displays trunk ports and allowed VLANs.
- `show port-security interface ...`: Shows port security status.
- `show etherchannel summary`: EtherChannel status.
- `show interfaces port-channel 1`: Details for EtherChannel interface.
- `show spanning-tree`: STP status.
- `show running-config interface vlan 99`: SVI config.
- `show ip interface brief`: IP addresses and status.
- `show interfaces [interface]`: Detailed interface info.
- `show interfaces description`: Interface descriptions.

---

## Notes

- Use `switchport mode trunk` for uplinks between switches.
- Use `switchport mode access` for end devices (computers, printers).
- For EtherChannel, configuration must match on both sides.
- Always verify configuration with the appropriate show commands.

