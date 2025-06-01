# ACL, NAT, and PAT Troubleshooting Guide

This guide covers key concepts, configuration, and troubleshooting steps for Access Control Lists (ACL), Network Address Translation (NAT), and Port Address Translation (PAT) on Cisco devices.

---

## ACL (Access Control List)

**ACLs are used to permit or deny traffic based on source/destination IP, protocol, or port. They are commonly used for security and traffic filtering.**

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
 ip address 192.168.1.10 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 203.0.113.2 255.255.255.0
 ip nat outside

ip nat inside source static 192.168.1.10 203.0.113.10
```
**Explanation:**
- `ip nat inside source static 192.168.1.10 203.0.113.10`: Maps internal IP 192.168.1.10 to public IP 203.0.113.10 one-to-one.

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

### Example
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
