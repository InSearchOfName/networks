# ACL, NAT, and PAT Troubleshooting Guide

This guide covers key concepts, configuration, and troubleshooting steps for Access Control Lists (ACL), Network Address Translation (NAT), and Port Address Translation (PAT) on Cisco devices.

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
