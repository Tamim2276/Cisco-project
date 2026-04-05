# E-Sports Tournament Network Project Plan (Cisco Packet Tracer)

## 1. Project Overview

This project simulates a full network for an E-Sports tournament venue. The design emphasizes low-latency communication, security isolation, mixed routing domains, and redundancy. It is structured to include all required networking topics:

1. Static Routing
2. VLAN
3. DHCP
4. EIGRP
5. OSPF
6. RIP
7. ACL
8. VLSM

---

## 2. Exact Topology Diagram Plan (Devices + Links)

### 2.1 Logical Topology

```text
Internet/Cloud
   |
  R5 (ISP)
   |
  R1 (Core)
 / |  \
R2 R3  R4
|  |   |
S2 S3  S4
|  |   |
Arena Ops Backup

Core services:
R1 --- S1 --- DMZ servers

Extra resiliency link:
R3 -------- R4  (backup WAN path)
```

### 2.2 Router Roles

1. **R1**: Core router and redistribution point, hosts centralized DHCP and the default route to the ISP
2. **R2**: Arena router, runs EIGRP, and provides the gateway for VLANs 10/20/50
3. **R3**: Operations router, runs RIP v2, and provides the gateway for VLANs 30/40/80
4. **R4**: Backup router, runs OSPF, and provides the gateway for VLAN 90
5. **R5**: ISP edge router with the return route back to the enterprise network

---

## 3. Exact Device Count

### 3.1 Network Infrastructure

1. Routers: 5
   - R1, R2, R3, R4, R5
2. Switches: 4
   - S1 (Core Services), S2 (Arena), S3 (Operations), S4 (Backup)

### 3.2 Servers

1. SRV1 Tournament Web
2. SRV2 Ticketing API
3. SRV3 Stream Relay
4. SRV4 NMS/Syslog

Total Servers: 4

### 3.2.1 Server Roles and Purpose

1. **SRV1 Tournament Web (10.77.2.98)**
   - Hosts tournament web/front-end content.
   - Purpose: Demonstrates DMZ-hosted public service access from authorized users.
2. **SRV2 Ticketing API (10.77.2.99)**
   - Represents ticketing/backend API service.
   - Purpose: Demonstrates business application reachability and segmentation in DMZ.
3. **SRV3 Stream Relay (10.77.2.100)**
   - Represents stream/media relay service.
   - Purpose: Demonstrates service availability for event broadcast traffic.
4. **SRV4 NMS/Syslog (10.77.2.66)**
   - Management and logging endpoint on the management subnet.
   - Purpose: Demonstrates operational monitoring separation from user networks.

### 3.3 End Devices

1. Player PCs: 1
2. Caster PCs: 1
3. Guest PCs: 1
4. Operations PCs: 1
5. Admin PCs: 1
6. Legacy devices: 1
7. Backup site server/PC: 1

Total End Devices: 7

### 3.4 Grand Total

1. Infrastructure devices (routers + switches): 9
2. Servers: 4
3. End devices: 7

**Grand Total: 20 devices**

---

## 4. Complete IP Addressing + VLSM Table

### 4.1 Addressing Strategy

- Parent network block: **10.77.0.0/16**
- VLSM applied by allocating larger host-demand VLANs first (Guests, Players), then smaller operational segments.

### 4.2 VLAN/Subnet Allocation

| Segment           | VLAN | Hosts Needed | Subnet         | Mask            | Default Gateway | DHCP Range             |
| ----------------- | ---: | -----------: | -------------- | --------------- | --------------- | ---------------------- |
| Guest users       |   50 |            1 | 10.77.0.0/24   | 255.255.255.0   | 10.77.0.1       | DHCP on R1             |
| Players           |   10 |            3 | 10.77.1.0/25   | 255.255.255.128 | 10.77.1.1       | DHCP on R1             |
| Backup site users |   90 |            1 | 10.77.1.128/26 | 255.255.255.192 | 10.77.1.129     | DHCP on R1             |
| Casters           |   20 |            1 | 10.77.1.192/26 | 255.255.255.192 | 10.77.1.193     | DHCP on R1             |
| Operations        |   30 |            1 | 10.77.2.0/27   | 255.255.255.224 | 10.77.2.1       | DHCP on R1             |
| Admin             |   40 |            1 | 10.77.2.32/27  | 255.255.255.224 | 10.77.2.33      | DHCP on R1             |
| Management        |   70 |            4 | 10.77.2.64/27  | 255.255.255.224 | 10.77.2.65      | Static / server access |
| DMZ Servers       |   60 |            4 | 10.77.2.96/28  | 255.255.255.240 | 10.77.2.97      | Static only            |
| Legacy            |   80 |            1 | 10.77.2.112/28 | 255.255.255.240 | 10.77.2.113     | DHCP on R1             |

### 4.3 WAN /30 Links

| Link           | Subnet          | Side A           | Side B           |
| -------------- | --------------- | ---------------- | ---------------- |
| R1-R2          | 10.77.255.0/30  | R1: 10.77.255.1  | R2: 10.77.255.2  |
| R1-R3          | 10.77.255.4/30  | R1: 10.77.255.5  | R3: 10.77.255.6  |
| R1-R4          | 10.77.255.8/30  | R1: 10.77.255.9  | R4: 10.77.255.10 |
| R1-R5          | 10.77.255.12/30 | R1: 10.77.255.13 | R5: 10.77.255.14 |
| R3-R4 (backup) | 10.77.255.16/30 | R3: 10.77.255.17 | R4: 10.77.255.18 |

### 4.4 Static Server IP Plan (DMZ)

1. SRV1 Web: 10.77.2.98
2. SRV2 Ticketing: 10.77.2.99
3. SRV3 Stream: 10.77.2.100
4. SRV4 NMS/Syslog (Management VLAN 70): 10.77.2.66

---

## 5. Exact Cable Map

### 5.1 Cable Types

1. **Serial DCE/DTE**: Router-to-router WAN links
2. **Copper Straight-Through**: Router-to-switch and host-to-switch links

### 5.2 WAN Cabling

1. R1 Se0/3/0 <-> R2 Se0/3/0 (Serial)
2. R1 Se0/3/1 <-> R3 Se0/3/0 (Serial)
3. R1 Se0/2/0 <-> R4 Se0/3/0 (Serial)
4. R1 Se0/2/1 <-> R5 Se0/3/0 (Serial)
5. R3 Se0/3/1 <-> R4 Se0/3/1 (Serial, backup path)

### 5.3 LAN/Uplink Cabling

1. R1 G0/0 <-> S1 G0/1 (Trunk)
2. R2 G0/0 <-> S2 G0/1 (Trunk)
3. R3 G0/0 <-> S3 G0/1 (Trunk)
4. R4 G0/0 <-> S4 G0/1 (Access or single-VLAN uplink)

### 5.4 Server Cabling (to S1)

1. S1 F0/2 -> SRV1
2. S1 F0/3 -> SRV2
3. S1 F0/4 -> SRV3
4. S1 F0/5 -> SRV4

### 5.5 User Access Cabling

1. S2 access ports for VLAN 10, 20, 50 user endpoints
2. S3 access ports for VLAN 30, 40, 80 user endpoints
3. S4 access ports for VLAN 90 backup-site endpoints

---

## 6. VLAN Plan

| VLAN ID | Name         | Subnet         |
| ------: | ------------ | -------------- |
|      10 | Players      | 10.77.1.0/25   |
|      20 | Casters      | 10.77.1.192/26 |
|      30 | Operations   | 10.77.2.0/27   |
|      40 | Admin        | 10.77.2.32/27  |
|      50 | Guests       | 10.77.0.0/24   |
|      60 | DMZ Servers  | 10.77.2.96/28  |
|      70 | Management   | 10.77.2.64/27  |
|      80 | Legacy       | 10.77.2.112/28 |
|      90 | Backup Users | 10.77.1.128/26 |

---

## 7. Router-by-Router Command Checklist (Packet Tracer)

> Note: Interface names may differ by router model in Packet Tracer. Use equivalent interfaces if needed.

### 7.1 R1 (Core-ASBR) Checklist

1. Basic setup
   - hostname, secure access, centralized policy
2. Interface IP configuration
   - Configure serial /30 links to R2, R3, R4, R5
   - `no shutdown` on all active interfaces
   - Set `clock rate` on DCE serial ends (if required)
3. Router-on-a-stick subinterfaces on G0/0
   - G0/0.60 -> dot1q 60, IP 10.77.2.97/28
   - G0/0.70 -> dot1q 70, IP 10.77.2.65/27
4. Centralized DHCP server
   - Excluded addresses
   - Pools for VLANs 10, 20, 30, 40, 50, 80, 90
   - Set default-router and DNS per pool
5. OSPF configuration
   - `router ospf 10`
   - Advertise R1-R4 link + VLAN 60/70
6. EIGRP configuration
   - `router eigrp 100`
   - `no auto-summary`
   - Advertise R1-R2 link
7. RIP configuration
   - `router rip`
   - `version 2`
   - `no auto-summary`
   - Advertise R1-R3 link
8. Route redistribution (critical)
   - Redistribute EIGRP and RIP into OSPF
   - Redistribute OSPF and RIP into EIGRP (with metrics)
   - Redistribute OSPF and EIGRP into RIP (with metric)
9. Static default route
   - `ip route 0.0.0.0 0.0.0.0 10.77.255.14`
10. Verification

- `show ip route`
- `show ip protocols`
- `show ip ospf neighbor`
- `show ip eigrp neighbors`
- `show ip rip database`

### 7.2 R2 (Arena Router - EIGRP Domain) Checklist

1. Interface IPs
   - Se0/3/0 -> 10.77.255.2/30
   - G0/0 physical up
2. Subinterfaces / VLAN gateways
   - G0/0.10 -> 10.77.1.1/25
   - G0/0.20 -> 10.77.1.193/26
   - G0/0.50 -> 10.77.0.1/24
3. DHCP relay on user VLANs
   - `ip helper-address 10.77.255.1`
4. EIGRP config
   - `router eigrp 100`
   - `no auto-summary`
   - Advertise local VLAN subnets + R1 link
5. ACL policy for guest isolation
   - Deny Guest -> Admin subnet
   - Permit Guest -> DMZ services that are required
   - Permit other needed traffic
   - Apply ACL inbound on VLAN 50 interface
6. Verification
   - `show access-lists`
   - `show ip eigrp neighbors`
   - End-to-end ping tests

### 7.3 R3 (Operations Router - RIP Domain) Checklist

1. Interface IPs
   - Se0/3/0 -> 10.77.255.6/30
   - Se0/3/1 -> 10.77.255.17/30
   - G0/0 physical up
2. Subinterfaces / VLAN gateways
   - G0/0.30 -> 10.77.2.1/27
   - G0/0.40 -> 10.77.2.33/27
   - G0/0.80 -> 10.77.2.113/28
3. DHCP relay
   - `ip helper-address 10.77.255.1` on VLAN 30/40/80
4. RIP v2
   - `router rip`
   - `version 2`
   - `no auto-summary`
   - Advertise local subnets + serial links
5. Optional ACL hardening
   - Additional restrictions for Admin segment
6. Verification
   - `show ip route rip`
   - `show ip rip database`

### 7.4 R4 (Backup Router - OSPF Domain) Checklist

1. Interface IPs
   - Se0/3/0 -> 10.77.255.10/30
   - Se0/3/1 -> 10.77.255.18/30
   - G0/0 -> 10.77.1.129/26
2. DHCP relay
   - `ip helper-address 10.77.255.1` on G0/0
3. OSPF
   - `router ospf 10`
   - Advertise backup LAN + both serial links
4. Verification
   - `show ip ospf neighbor`
   - `show ip route ospf`

### 7.5 R5 (ISP Router) Checklist

1. Interface IP
   - Se0/3/0 -> 10.77.255.14/30
2. Static return route
   - `ip route 10.77.0.0 255.255.0.0 10.77.255.13`
3. Optional internet simulation
   - Attach additional cloud/public segment if desired
4. Verification
   - Ping R1 and internal reachable addresses

---

## 8. Switch Configuration Checklist

### 8.1 S1 (Core Services)

1. Create VLAN 60 and VLAN 70
2. Configure trunk port to R1, allow VLAN 60,70
3. Assign server ports to VLAN 60
4. Assign NMS/Syslog port to VLAN 70

### 8.2 S2 (Arena)

1. Create VLAN 10, 20, 50
2. Configure trunk to R2, allow VLAN 10,20,50
3. Assign access ports:
   - Players -> VLAN 10
   - Casters -> VLAN 20
   - Guests -> VLAN 50

### 8.3 S3 (Operations)

1. Create VLAN 30, 40, 80
2. Configure trunk to R3, allow VLAN 30,40,80
3. Assign access ports by department

### 8.4 S4 (Backup)

1. Create VLAN 90
2. Assign backup user ports to VLAN 90
3. Configure uplink to R4 on G0/1
4. Connect the backup server/PC to Fa0/1

---

## 9. Requirement Coverage Mapping

| Required Topic | Where Implemented                                                     |
| -------------- | --------------------------------------------------------------------- |
| Static Routing | R1 default route to R5, R5 static return route                        |
| VLAN           | VLANs 10/20/30/40/50/60/70/80/90 on switches and router subinterfaces |
| DHCP           | Centralized DHCP pools on R1 + helper-address on R2/R3/R4             |
| EIGRP          | Arena domain between R1-R2 and associated VLAN networks               |
| OSPF           | Core/backup domain between R1-R4 (+ service VLANs on R1)              |
| RIP            | Operations/legacy domain on R3 and R1-R3 link                         |
| ACL            | Guest isolation and controlled access to Admin/DMZ                    |
| VLSM           | Variable subnet sizes across all LAN segments and /30 WAN links       |

---

## 9.1 ACL Purpose and Usage in This Project

### What ACL Is Doing

In this topology, ACL is used as a security filter for Guest users.

1. It blocks Guest VLAN traffic to sensitive internal networks (Admin, Operations, Management, Legacy).
2. It allows approved traffic to required services (for example DMZ targets).
3. It allows non-restricted traffic to continue toward upstream destinations.

### Where ACL Is Applied

1. Device: `R2_Arena`
2. ACL name: `GUEST_ISOLATION`
3. Interface: `g0/0.50` (Guest VLAN subinterface)
4. Direction: inbound (`ip access-group GUEST_ISOLATION in`)

### Why Inbound on Guest Interface

1. Guest traffic is filtered immediately when it enters the router.
2. Unauthorized traffic is dropped before it reaches protected networks.
3. This reduces risk and unnecessary traffic in the core.

### How to Demonstrate ACL Live

From Guest PC:

1. `ping 10.77.2.33` -> should fail (Admin blocked)
2. `ping 10.77.2.1` -> should fail (Operations blocked)
3. `ping 10.77.2.98` -> should pass (allowed DMZ target)

On R2:

1. `show access-lists GUEST_ISOLATION`

Expected proof:

1. Deny and permit lines show increasing match counters.

---

## 10. Final Verification Checklist (Demo Day)

1. Test DHCP leases for Players, Guests, Admin, Backup clients
2. Test inter-VLAN routing for authorized users
3. Verify ACL blocks Guest -> Admin
4. Verify Guest can access allowed DMZ services
5. Check routing tables on all routers
6. Simulate WAN failure (disable one serial link) and verify alternate path behavior
7. Save configs (`copy running-config startup-config`) on all routers and switches

---

## 11. Suggested File Naming for Submission

- Packet Tracer file: `esports_tournament_network.pkt`
- Report: `esports_tournament_network_report.pdf`
- Presentation: `esports_tournament_network_presentation.pptx`
- Zip: `GroupMemberNames_EsportsTournamentNetwork.zip`
