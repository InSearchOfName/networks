# TL;DR: Cisco Networking Theory Cheat Sheet

## VLANs
- VLANs segment a network into separate broadcast domains.
- Use access ports for end devices, trunk ports for switch uplinks.
- Port security restricts which devices can connect to a port.

## Trunking
- Trunks carry multiple VLANs between switches.
- Native VLAN is untagged; all others are tagged.

## EtherChannel (LAG)
- Bundles multiple physical links into one logical link for redundancy and bandwidth.
- Must match config on both sides.
- LACP (active/passive) or static (on).

## Spanning Tree Protocol (STP)
- Prevents loops in switched networks by blocking redundant paths.
- Rapid PVST+ is a Cisco fast-convergence version.
- Root bridge is the logical center of the network.

## Switch Management
- Assign an IP to a VLAN interface (SVI) for remote management.
- Set a default gateway for out-of-subnet management.

## Duplex & Speed
- Full duplex: send/receive at the same time (preferred).
- Half duplex: send or receive, not both.
- Set speed/duplex manually if auto-negotiation fails.

## ACL (Access Control List)
- Filters traffic based on IP, protocol, port.
- Implicit deny at the end.
- Applied inbound or outbound on interfaces.

## NAT (Network Address Translation)
- Translates private IPs to public IPs for internet access.
- Inside = LAN, Outside = WAN.
- NAT pool: range of public IPs.

## PAT (Port Address Translation)
- NAT overload: many devices share one public IP using different ports.

## DHCP Relay
- Forwards DHCP requests to a server on another subnet using `ip helper-address`.

## HSRP (Hot Standby Router Protocol)
- Provides gateway redundancy.
- Routers share a virtual IP; one is active, others standby.

## Static Routes
- Default route (`0.0.0.0/0`) sends all unknown traffic to a next-hop.

## Loopback Interface
- Virtual interface always up; used for router IDs, management, testing.

## Router-on-a-Stick
- One physical interface, multiple subinterfaces (one per VLAN) for inter-VLAN routing.

## Show Commands (Verification)
- `show vlan brief`, `show interfaces status`, `show interfaces trunk`
- `show etherchannel summary`, `show spanning-tree`
- `show ip interface brief`, `show running-config`
- `show access-lists`, `show ip nat translations`, `show standby`
- `show ip route`, `show interfaces [interface]`

---
**Tip:** Always verify configuration and status with the appropriate `show` commands.
