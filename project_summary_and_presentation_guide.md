# E-Sports Tournament Network: Project Summary and Presentation Guide

## 0. Project Idea (Very Easy Explanation)

You can explain the idea like this to your teacher:

"I built a tournament network where every group has its own lane.

1. Players, casters, operations, admin, guests, servers, and backup users are separated into VLANs.
2. Routers connect all sites, but security rules control who can access what.
3. One core router gives IP addresses (DHCP) to all branch users.
4. Different routing protocols are used in different sites (EIGRP, RIP, OSPF), and the core router connects them together using redistribution.
5. A guest ACL blocks access to sensitive internal networks but allows limited allowed traffic.
6. The ISP router gives upstream connectivity and a return path.

So this project proves segmentation, routing, DHCP, ACL security, and VLSM in one complete design."

---

## 1. What I Built

This project is a complete multi-site enterprise network in Cisco Packet Tracer for an E-Sports tournament environment.

It includes:

1. Core site with DMZ and Management VLANs
2. Arena site with Players, Casters, and Guest VLANs
3. Operations site with Ops, Admin, and Legacy VLANs
4. Backup site with a dedicated subnet
5. ISP edge router connection
6. Mixed dynamic routing with redistribution (EIGRP, OSPF, RIP)
7. Centralized DHCP server
8. ACL-based traffic control for Guest users
9. VLSM-based subnet design

---

## 2. Final Topology Overview

### Routers

1. R1_Core (central core/distribution router)
2. R2_Arena (EIGRP domain branch)
3. R3_Operations (RIP domain branch)
4. R4_Backup (OSPF branch + resiliency link)
5. R5_ISP (ISP edge)

### Switches

1. S1_Core (DMZ + Management servers)
2. S2_Arena (VLAN 10, 20, 50)
3. S3_Operations (VLAN 30, 40, 80)
4. S4_Backup (VLAN 90 access)

### WAN Links (/30)

1. R1-R2: 10.77.255.0/30
2. R1-R3: 10.77.255.4/30
3. R1-R4: 10.77.255.8/30
4. R1-R5: 10.77.255.12/30
5. R3-R4 backup link: 10.77.255.16/30

---

## 3. What the Network Is Doing (Functionality)

## 3.1 VLAN Segmentation

The network separates users/services into VLANs to improve security and control broadcast domains.

1. VLAN 10: Players
2. VLAN 20: Casters
3. VLAN 30: Operations
4. VLAN 40: Admin
5. VLAN 50: Guests
6. VLAN 60: DMZ Servers
7. VLAN 70: Management/Syslog
8. VLAN 80: Legacy
9. VLAN 90: Backup users

## 3.2 Inter-VLAN Routing

Router-on-a-stick subinterfaces provide default gateways for VLANs.

1. R1 handles VLAN 60 and 70
2. R2 handles VLAN 10, 20, 50
3. R3 handles VLAN 30, 40, 80
4. R4 handles VLAN 90 (single LAN interface)

## 3.3 DHCP (Centralized)

R1 provides DHCP pools for all user VLANs.

Branch routers (R2, R3, R4) forward broadcast DHCP requests to R1 using `ip helper-address`.

Result: End devices automatically receive IP, subnet mask, gateway, and DNS.

## 3.4 Dynamic Routing

Different protocols are used by design requirement:

1. EIGRP on Arena side
2. RIP v2 on Operations side
3. OSPF on Core/Backup side

R1 redistributes routes between these protocols, so all sites can communicate end-to-end.

## 3.5 Static Routing

1. R1 has default route to ISP (R5)
2. R5 has static return summary route to enterprise network 10.77.0.0/16

This provides complete two-way upstream/downstream reachability.

## 3.6 ACL Security Policy

An ACL on R2 Guest VLAN subinterface blocks Guests from sensitive internal VLANs and allows controlled access.

Blocked examples:

1. Guest -> Admin VLAN
2. Guest -> Operations VLAN
3. Guest -> Management VLAN
4. Guest -> Legacy VLAN

Allowed examples:

1. Guest -> DMZ target(s)
2. Guest -> upstream/other permitted traffic

## 3.7 VLSM

Subnets are allocated by need, for efficient address use.

Examples:

1. /24 for guest network
2. /25, /26 for larger user segments
3. /27, /28 for smaller segments and DMZ
4. /30 for WAN point-to-point links

---

## 4. Evidence of Completion

The project has been validated with:

1. Successful DHCP leases on branch and backup endpoints
2. Inter-site pings (including cross-protocol paths)
3. R5 to R1 connectivity checks
4. OSPF/EIGRP/RIP learned routes in routing table
5. ACL behavior tests (deny and permit cases)

---

## 4.1 Functionality Test Matrix (Command + Where to Run)

Use this section during demo so you can show each required topic with proof.

### A. Static Routing

Where to run:

1. R1_Core
2. R5_ISP

Commands:

1. On R1_Core: `show ip route | include 0.0.0.0`
2. On R5_ISP: `show ip route | include 10.77.0.0`
3. On R5_ISP: `ping 10.77.255.13`

Expected:

1. R1 has default route via 10.77.255.14
2. R5 has return route to 10.77.0.0/16 via 10.77.255.13
3. Ping succeeds

### B. VLAN

Where to run:

1. S1_Core, S2_Arena, S3_Operations

Commands:

1. `show vlan brief`
2. `show interfaces trunk`

Expected:

1. VLANs appear correctly
2. G0/1 trunk is up toward router

### C. DHCP

Where to run:

1. End device Desktop > IP Configuration
2. R1_Core

Commands:

1. On clients: switch to DHCP and confirm leased address
2. On R1_Core: `show ip dhcp binding`

Expected:

1. Clients get IP/gateway automatically
2. Bindings appear on R1

### D. EIGRP

Where to run:

1. R1_Core and R2_Arena

Commands:

1. `show ip eigrp neighbors`
2. `show ip route eigrp`

Expected:

1. R1-R2 EIGRP neighbor is up
2. EIGRP routes are present

### E. OSPF

Where to run:

1. R1_Core and R4_Backup

Commands:

1. `show ip ospf neighbor`
2. `show ip route ospf`

Expected:

1. OSPF neighbor is up
2. OSPF routes are present

### F. RIP

Where to run:

1. R1_Core and R3_Operations

Commands:

1. `show ip route rip`
2. `show ip protocols`

Expected:

1. RIP-learned routes exist
2. RIP process is active with version 2

### G. ACL

Where to run:

1. Guest PC
2. R2_Arena

Commands:

1. Guest PC: `ping 10.77.2.33` (should fail)
2. Guest PC: `ping 10.77.2.98` (should pass)
3. R2_Arena: `show access-lists GUEST_ISOLATION`
4. R2_Arena: `show ip interface g0/0.50`

Expected:

1. Blocked and allowed behavior matches policy
2. ACL counters increase

### H. VLSM

Where to show:

1. Use subnet table in this document
2. Optionally show one example from router interfaces

Commands:

1. On R1/R2/R3/R4: `show ip interface brief`

Expected:

1. Mixed subnet masks (/24, /25, /26, /27, /28, /30) are visible and correct

---

## 5. How to Present to Teacher (Step-by-Step Demo)

Follow this exact order during demo.

## 5.1 Start with Architecture (1 minute)

Explain:

1. Why network is split into sites (Core, Arena, Operations, Backup, ISP)
2. Why VLAN segmentation is used
3. Why mixed routing protocols were required

Show the topology map and identify each router/switch role.

## 5.2 Show IP Plan + VLSM (1 minute)

Explain:

1. Parent block is 10.77.0.0/16
2. Different subnet masks were chosen based on host demand
3. WAN uses /30 point-to-point links

## 5.3 Show Routing Functionality (2 minutes)

On R1:

1. `show ip route`
2. `show ip protocols`

Explain:

1. Which routes are learned by EIGRP, OSPF, RIP
2. Why redistribution is needed at R1

## 5.4 Show DHCP Functionality (1 minute)

On one client from Arena/Operations/Backup:

1. Show IP config obtained by DHCP
2. Mention helper-address relays request to R1

## 5.5 Show Inter-VLAN and Inter-Site Reachability (1 minute)

Run pings:

1. Local gateway ping
2. Cross-site ping
3. DMZ server ping

Explain first-ping timeout may occur due to ARP and is normal.

## 5.6 Show ACL Security Control (2 minutes)

From Guest PC:

1. Ping Admin subnet target (must fail)
2. Ping allowed DMZ target (must pass)

On R2:

1. `show access-lists GUEST_ISOLATION`
2. `show ip interface g0/0.50`

Explain ACL is applied inbound on Guest VLAN subinterface.

## 5.7 Show Static Route + ISP Return Path (1 minute)

On R1:

1. Show default route to R5

On R5:

1. Show static route back to 10.77.0.0/16
2. Ping R1 edge IP

Explain why both outbound and return routes are required.

## 5.8 Close with Save/Operational Readiness (30 sec)

Run on device:

1. `copy running-config startup-config`

Explain this makes config persistent after reboot/power loss.

---

## 6. Viva/Question Answers (Quick)

### Q1: Why use three routing protocols?

Because project requirements asked to demonstrate EIGRP, OSPF, and RIP. R1 redistributes them so all networks are reachable.

### Q2: Why ACL on Guest VLAN?

To enforce least-privilege access. Guests should not reach sensitive internal VLANs.

### Q3: Why centralized DHCP?

Single control point on R1 simplifies management; helper-address allows remote VLANs to still use it.

### Q4: Why VLSM?

To avoid wasting IP space and size each subnet based on real host count.

---

## 7. Final Submission Checklist

1. Packet Tracer file saved
2. All devices config saved to startup-config
3. Routing verification screenshots
4. DHCP success screenshots
5. ACL pass/fail proof screenshots
6. Project plan/report updated to final topology
7. Presentation demo order rehearsed

---

## 8. Server Roles and How to Show Their Functionality

## 8.1 What Each Server Is Doing

1. `SRV1_Web (10.77.2.98)`
   - Hosts tournament web/front-end service
   - Main DMZ reachability proof target
2. `SRV2_Ticket (10.77.2.99)`
   - Represents ticketing/API backend service
   - Second business service in DMZ
3. `SRV3_Stream (10.77.2.100)`
   - Stream relay/media service
   - Also used as NTP source in your project setup
4. `SRV4_Syslog (10.77.2.66)`
   - Management/logging server on management subnet
   - Shows separation between DMZ and management traffic
5. `Server5 (Backup subnet endpoint)`
   - Backup-site server/endpoint in VLAN 90 network
   - Proves backup-site DHCP and cross-site reachability

## 8.2 Live Demo Commands (Teacher Presentation)

### A. Show Server IP Roles

On each server: Desktop > IP Configuration

1. Show SRV1/SRV2/SRV3 are in DMZ-related service segment
2. Show SRV4 is in management subnet (`10.77.2.66`)
3. Show Server5 is in backup subnet (`10.77.1.128/26`)

### B. Show DMZ Service Reachability

From an Arena or Guest client Command Prompt:

1. `ping 10.77.2.98`
2. `ping 10.77.2.99`
3. `ping 10.77.2.100`

Expected:

1. Pings to allowed services succeed (first ping may time out due to ARP)

### C. Show Security Separation

From Guest PC:

1. `ping 10.77.2.66`

Expected:

1. Based on ACL policy, management/internal restricted targets should be blocked

### D. Show Backup-Site Functionality

On `Server5` Command Prompt:

1. `ipconfig`
2. `ping 10.77.1.129`
3. `ping 10.77.2.98`

Expected:

1. DHCP lease from backup subnet is present
2. Gateway is reachable
3. Cross-site service access works

### E. Show Router Evidence

1. On `R1_Core`: `show ip route`
2. On `R2_Arena`: `show access-lists GUEST_ISOLATION`

Expected:

1. Multi-protocol routes are visible
2. ACL counters increase during tests

## 8.3 One-Line Teacher Explanation

"These servers simulate real tournament services: web, ticketing/API, stream relay, management logging, and backup endpoint. I proved their functionality using selective reachability, ACL control, DHCP leasing, and cross-site routing tests."
