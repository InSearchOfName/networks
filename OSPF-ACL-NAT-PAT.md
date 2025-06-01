# ACL, NAT, PAT, and OSPF Troubleshooting Guide

This guide covers key concepts, configuration, and troubleshooting steps for Access Control Lists (ACL), Network Address Translation (NAT), Port Address Translation (PAT), and OSPF on Cisco devices.

---

## ACL (Access Control List)

**ACLs are used to permit or deny traffic based on source/destination IP, protocol, or port. They are commonly used for security and traffic filtering.**

### Types of ACLs

- **Numbered ACLs:** Use numbers to identify the ACL (e.g., 1-99 for standard, 100-199 for extended).
- **Named ACLs:** Use a custom name to identify the ACL, allowing easier editing and management.

### Numbered ACL Example

```
access-list 100 permit <protocol> <source-ip> <wildcard-mask> any eq <port>
interface GigabitEthernet0/0
 ip access-group 100 in
```
**Example (permit HTTP from 192.168.1.0/24):**
```
access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80
interface GigabitEthernet0/0
 ip access-group 100 in
```
**Explanation:**
- `access-list 100 permit <protocol> <ip> <wildcard-mask> any eq <port>`: Permits specified protocol and port from the given source IP/subnet to any destination.
- `ip access-group 100 in`: Applies ACL 100 inbound on the interface.

### Named ACL Example

```
ip access-list extended <ACL-NAME>
 permit <protocol> <source-ip> <wildcard-mask> any eq <port>
 deny ip any any
interface GigabitEthernet0/0
 ip access-group <ACL-NAME> in
```
**Example (permit HTTP from 192.168.1.0/24):**
```
ip access-list extended WEB-ONLY
 permit tcp 192.168.1.0 0.0.0.255 any eq 80
 deny ip any any
interface GigabitEthernet0/0
 ip access-group WEB-ONLY in
```
**Explanation:**
- `ip access-list extended <ACL-NAME>`: Creates a named extended ACL.
- `permit <protocol> <ip> <wildcard-mask> any eq <port>`: Permits specified protocol and port from the given source IP/subnet.
- `deny ip any any`: Explicitly denies all other traffic (optional, as there is an implicit deny).
- `ip access-group <ACL-NAME> in`: Applies the named ACL inbound.

### Common Issues
- ACL applied in the wrong direction or interface.
- Implicit deny at the end of every ACL.
- Incorrect sequence/order of ACL statements.
- Missing permit for required traffic.

### Troubleshooting Steps
1. **Check ACL configuration:**
    ```
    show access-lists
    show running-config | include access-list
    ```
2. **Check ACL application:**
    ```
    show running-config interface [interface]
    ```
3. **Check hit counts:**
    ```
    show access-lists
    ```
    - Look for incrementing counters to verify if traffic matches the ACL.

4. **Verify interface direction:**
    - `in` = traffic entering the interface.
    - `out` = traffic leaving the interface.

### Example
```
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
interface GigabitEthernet0/0
 ip access-group 100 in
```

---

## NAT (Network Address Translation)

**NAT translates private IP addresses to public IP addresses for internet access.**

### Types of NAT
- **Dynamic NAT:** Maps private IPs to a pool of public IPs.
- **PAT (NAT Overload):** Maps many private IPs to one public IP using different ports.
- **Static NAT:** Maps a single private IP to a single public IP (one-to-one mapping).

### Common Issues
- NAT not applied to correct interfaces (`ip nat inside`/`ip nat outside`).
- Missing or incorrect NAT rules.
- Overlapping or missing access lists.
- NAT pool exhausted.
- Static NAT entry missing or incorrect.

### Troubleshooting Steps
1. **Check NAT configuration:**
    ```
    show running-config | include nat
    ```
2. **Check NAT translations:**
    ```
    show ip nat translations
    ```
3. **Check NAT statistics:**
    ```
    show ip nat statistics
    ```
4. **Verify interface roles:**
    ```
    show running-config interface [interface]
    ```
    - Ensure correct `ip nat inside` and `ip nat outside` assignments.

### Dynamic NAT Example
```
interface GigabitEthernet0/0
 ip address <inside-ip> <subnet-mask>
 ip nat inside

interface GigabitEthernet0/1
 ip address <outside-ip> <subnet-mask>
 ip nat outside

ip nat pool <POOLNAME> <start-public-ip> <end-public-ip> netmask <netmask>
access-list 1 permit <inside-subnet> <wildcard-mask>
ip nat inside source list 1 pool <POOLNAME>
```
**Example:**
```
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 203.0.113.2 255.255.255.0
 ip nat outside

ip nat pool MYPOOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 pool MYPOOL
```

### Static NAT Example
```
interface GigabitEthernet0/0
 ip address <inside-ip> <subnet-mask>
 ip nat inside

interface GigabitEthernet0/1
 ip address <outside-ip> <subnet-mask>
 ip nat outside

ip nat inside source static <inside-ip> <public-ip>
```
**Example:**
```
interface GigabitEthernet0/0
 ip address 192.168.1.10 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 203.0.113.2 255.255.255.0
 ip nat outside

ip nat inside source static 192.168.1.10 203.0.113.10
```
**Explanation:**
- `ip nat inside source static <inside-ip> <public-ip>`: Maps internal IP to public IP one-to-one.

---

## PAT (Port Address Translation / NAT Overload)

**PAT allows multiple devices to share a single public IP by translating source ports.**

### Common Issues
- PAT not configured on the correct interface.
- Access list does not match internal traffic.
- Overload keyword missing.

### Troubleshooting Steps
1. **Check PAT configuration:**
    ```
    show running-config | include nat
    ```
2. **Check translations:**
    ```
    show ip nat translations
    ```
3. **Check statistics:**
    ```
    show ip nat statistics
    ```
4. **Verify access-list and interface:**
    ```
    show access-lists
    show running-config interface [interface]
    ```

### PAT Example

```
access-list 1 permit <inside-subnet> <wildcard-mask>
ip nat inside source list 1 interface <outside-interface> overload
```
**Example:**
```
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

---

## OSPF (Open Shortest Path First)

**OSPF is a link-state routing protocol used for dynamic routing within an Autonomous System.**

### Key Concepts
- OSPF uses areas to optimize and organize routing (Area 0 is the backbone).
- Each router has a unique Router ID (RID).
- OSPF uses cost (based on bandwidth) as its metric.
- Neighbors must be in the same area and have matching parameters to form adjacencies.

### Common Issues
- Mismatched OSPF area numbers.
- Incorrect or missing network statements.
- Interface not enabled for OSPF.
- Router IDs not unique.
- Passive interfaces blocking OSPF hellos.
- Authentication mismatches (if used).

### Troubleshooting Steps
1. **Check OSPF configuration:**
    ```
    show running-config | section ospf
    ```
2. **Check OSPF neighbor relationships:**
    ```
    show ip ospf neighbor
    ```
3. **Check OSPF interface status:**
    ```
    show ip ospf interface
    ```
4. **Check OSPF routes in the routing table:**
    ```
    show ip route ospf
    ```
5. **Check OSPF process and database:**
    ```
    show ip ospf
    show ip ospf database
    ```
6. **Check passive interfaces:**
    ```
    show running-config | include passive-interface
    ```

### OSPF Example Configuration

```
router ospf 10
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 network 10.1.1.4 0.0.0.3 area 0
 passive-interface g0/0/0
```
**Explanation:**
- `router ospf 10`: Starts OSPF process 10.
- `router-id 1.1.1.1`: Sets the router's OSPF ID.
- `network ... area ...`: Advertises networks in OSPF area 0.
- `passive-interface ...`: Prevents OSPF hellos on the specified interface (no neighbor formed).

### OSPF Interface Example

```
interface g0/0/0
 ip ospf 10 area 0
```
**Explanation:**
- Enables OSPF process 10 on the interface in area 0 (interface-based OSPF configuration).

### General OSPF Troubleshooting Checklist

- Are all routers in the same area for a given subnet?
- Are network statements or interface OSPF commands correct?
- Are router IDs unique?
- Are interfaces up and not passive (unless intended)?
- Are OSPF neighbors established (`show ip ospf neighbor`)?
- Are OSPF routes present in the routing table?

### Useful OSPF Show Commands

```
show ip ospf
show ip ospf neighbor
show ip ospf interface
show ip ospf database
show ip route ospf
show running-config | section ospf
show running-config | include passive-interface
```

---

## General Troubleshooting Checklist

- Are the correct interfaces marked as `ip nat inside` and `ip nat outside`?
- Is the access-list permitting the correct source addresses?
- Are NAT/PAT rules applied to the correct interfaces?
- Are there NAT translations present (`show ip nat translations`)?
- Are hit counts increasing on ACLs?
- Is there an implicit deny blocking traffic in the ACL?
- Is the NAT pool exhausted?
- Is the overload keyword present for PAT?
- Are there any overlapping or conflicting NAT rules?

---

## Useful Show Commands

```
show access-lists
show running-config | include access-list
show running-config | include nat
show ip nat translations
show ip nat statistics
show running-config interface [interface]
debug ip nat
debug ip packet
```
**Note:** Use debug commands with caution in production environments.

---
