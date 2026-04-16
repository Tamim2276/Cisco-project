# Cisco Network Topology & ACL Demo Video Script

## 1. Introduction

Hello everyone! In this video, I'll walk you through our Cisco Packet Tracer network topology, explain how the network is structured, and demonstrate the functionality of our Access Control Lists (ACLs) by showing live ping tests from different PCs.

## 2. Topology Overview

Let's start with the network topology:

- We have a core router (R1_Core) connected to several branch routers, including R2_Arena, R3_Operations, R4_Backup, and R5_ISP.
- Each branch has its own VLANs and subnets. For example, the Arena site (R2_Arena) has VLANs for Players, Casters, and Guests, each with its own subnet.
- The DMZ hosts critical servers: the Tournament Web Server (10.77.2.98), Ticketing API Server (10.77.2.99), and Stream Server (10.77.2.100).
- Management and Syslog servers are on a separate management VLAN.
- All switches are configured with appropriate VLANs and trunk ports to their routers.

## 3. ACL Purpose and Placement

- ACLs are used to restrict access between VLANs and protect sensitive resources.
- For example, the CASTER_ACCESS_POLICY ACL on R2_Arena allows Casters full access to the Stream Server, but only HTTP/HTTPS to the Tournament Web and Ticketing API servers, while blocking access to management and internal networks.
- The PLAYER_STREAM_BLOCK ACL restricts Players to only Ticketing and Tournament Web on HTTP/HTTPS, blocking all other sensitive resources.
- The GUEST_ISOLATION ACL blocks Guests from all internal resources except for limited web access.

## 4. Demonstration: Ping Tests

Now, let's demonstrate the ACLs in action by performing ping tests from each PC:

### a) Player PC

- Ping 10.77.2.98 (Tournament Web): Should FAIL (ICMP blocked)
- Ping 10.77.2.99 (Ticketing API): Should FAIL (ICMP blocked)
- Ping 10.77.2.100 (Stream Server): Should FAIL (blocked by ACL)
- Ping 10.77.2.66 (Syslog): Should FAIL (blocked by ACL)
- https://10.77.2.98: Should PASS (HTTPS allowed)
- https://10.77.2.99: Should PASS (HTTPS allowed)
- https://10.77.2.100: Should Fail

### b) Caster PC

- Ping 10.77.2.98 (Tournament Web): Should FAIL (ICMP blocked)
- Ping 10.77.2.99 (Ticketing API): Should FAIL (ICMP blocked)
- Ping 10.77.2.100 (Stream Server): Should PASS (full access allowed)
- Ping 10.77.2.66 (Syslog): Should FAIL (blocked by ACL)
- https://10.77.2.98: Should PASS (HTTPS allowed)
- https://10.77.2.99: Should PASS (HTTPS allowed)

### c) Guest PC

- Ping to any internal server (e.g., 10.77.2.98, 10.77.2.66): Should FAIL (all blocked)
- Ping 10.77.2.98 (Tournament Web): Should FAIL (ICMP blocked)
- Ping 10.77.2.99 (Ticketing API): Should FAIL (ICMP blocked)
- Ping 10.77.2.100 (Stream Server): Should FAIL
- Ping 10.77.2.66 (Syslog): Should FAIL (blocked by ACL)
- Open browser to https://10.77.2.98: Should PASS (HTTPS allowed)
- Open browser to https://10.77.2.99: Should PASS (HTTPS allowed)
- Open browser to https://10.77.2.100: Should PASS (HTTPS allowed)
- Ping 10.77.2.34 Should fail (ADMIN)

## 5. Conclusion

This demonstration shows how ACLs can be used to tightly control access between different parts of the network, ensuring that only authorized traffic is permitted while protecting sensitive resources. Thank you for watching!
