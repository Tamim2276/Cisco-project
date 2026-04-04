# Full Router & Switch CLI Configurations (Ordered by Workflow)

Based on your sequence of building this locally (Core first, then branches: S1 -> Servers -> ISPs/Branches), here is the step-by-step CLI copy-paste guide customized for your specific interfaces (`Se0/2/1`, `g0/0`, etc.).

---

## 1. R1 Core Router (The center router)

```ios
enable
configure terminal
hostname R1_Core

! --- WAN Interfaces to the other routers ---
! To R5 (ISP)
interface Se0/2/1
 ip address 10.77.255.13 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! To R2 (Arena)
interface Se0/3/0
 ip address 10.77.255.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! To R3 (Ops)
interface Se0/3/1
 ip address 10.77.255.5 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! To R4 (Backup)
interface Se0/2/0
 ip address 10.77.255.9 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! --- LAN Interface (Router-on-a-Stick for DMZ and Mgmt) ---
interface g0/0
 no shutdown
 exit

interface g0/0.60
 encapsulation dot1Q 60
 ip address 10.77.2.97 255.255.255.240
 exit

interface g0/0.70
 encapsulation dot1Q 70
 ip address 10.77.2.65 255.255.255.224
 exit

! --- DHCP Server Pools ---
ip dhcp excluded-address 10.77.1.1
ip dhcp excluded-address 10.77.1.193
ip dhcp excluded-address 10.77.2.1
ip dhcp excluded-address 10.77.2.33
ip dhcp excluded-address 10.77.0.1
ip dhcp excluded-address 10.77.2.113
ip dhcp excluded-address 10.77.1.129

ip dhcp pool VLAN10_PLAYERS
 network 10.77.1.0 255.255.255.128
 default-router 10.77.1.1
 dns-server 8.8.8.8

ip dhcp pool VLAN20_CASTERS
 network 10.77.1.192 255.255.255.192
 default-router 10.77.1.193
 dns-server 8.8.8.8

ip dhcp pool VLAN50_GUESTS
 network 10.77.0.0 255.255.255.0
 default-router 10.77.0.1
 dns-server 8.8.8.8

ip dhcp pool VLAN30_OPS
 network 10.77.2.0 255.255.255.224
 default-router 10.77.2.1
 dns-server 8.8.8.8

ip dhcp pool VLAN40_ADMIN
 network 10.77.2.32 255.255.255.224
 default-router 10.77.2.33
 dns-server 8.8.8.8

ip dhcp pool VLAN80_LEGACY
 network 10.77.2.112 255.255.255.240
 default-router 10.77.2.113
 dns-server 8.8.8.8

ip dhcp pool VLAN90_BACKUP
 network 10.77.1.128 255.255.255.192
 default-router 10.77.1.129
 dns-server 8.8.8.8
 exit

! --- Routing Protocols & Redistribution ---
! Static route to internet
ip route 0.0.0.0 0.0.0.0 10.77.255.14

! OSPF (Core, Servers, and Backup Site)
router ospf 10
 network 10.77.2.96 0.0.0.15 area 0
 network 10.77.2.64 0.0.0.31 area 0
 network 10.77.255.8 0.0.0.3 area 0
 redistribute eigrp 100 subnets
 redistribute rip subnets
 default-information originate
 exit

! EIGRP (Arena Site)
router eigrp 100
 network 10.77.255.0 0.0.0.3
 redistribute ospf 10 metric 10000 100 255 1 1500
 redistribute rip metric 10000 100 255 1 1500
 exit

! RIP (Operations Site)
router rip
 version 2
 no auto-summary
 network 10.77.255.4
 redistribute ospf 10 metric 2
 redistribute eigrp 100 metric 2
 exit
```

---

## 2. S1 Core Switch (Below R1_Core)

Connect `R1_Core` Gig0/0 down to `S1` Gig0/1.

```ios
enable
configure terminal
hostname S1_Core

! Create the VLANs
vlan 60
 name DMZ_Servers
vlan 70
 name Management
 exit

! Configure the Trunk port pointing to the Router
interface g0/1
 switchport mode trunk
 exit

! Configure Access ports for DMZ Servers (VLAN 60)
interface range f0/2 - 4
 switchport mode access
 switchport access vlan 60
 exit

! Configure Access port for Syslog Server (VLAN 70)
interface f0/5
 switchport mode access
 switchport access vlan 70
 exit
```

---

## 3. Server Static IPs

Click each server -> **Desktop** -> **IP Configuration**, then set them statically:

- **Game Server (SRV1 on F0/2):**
  - IP Address: `10.77.2.98`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **Database Server (SRV2 on F0/3):**
  - IP Address: `10.77.2.99`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **NTP Server (SRV3 on F0/4):**
  - IP Address: `10.77.2.100`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **Syslog Server (SRV4 on F0/5):**
  - IP Address: `10.77.2.101`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

---

## 4. R5 ISP Router (Top Router)

```ios
enable
configure terminal
hostname R5_ISP

! Interface to R1 Core
interface Se0/3/0
 ip address 10.77.255.14 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! Static return route back to your enterprise network
ip route 10.77.0.0 255.255.0.0 10.77.255.13
```

---

## 5. R2 Arena Router (Left Router)

```ios
enable
configure terminal
hostname R2_Arena

! Interface to R1 Core
interface Se0/3/0
 ip address 10.77.255.2 255.255.255.252
 no shutdown
 exit

! Main LAN Interface
interface g0/0
 no shutdown
 exit

! Players VLAN Gateway
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.77.1.1 255.255.255.128
 ip helper-address 10.77.255.1
 exit

! Casters VLAN Gateway
interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.77.1.193 255.255.255.192
 ip helper-address 10.77.255.1
 exit

! Guests VLAN Gateway
interface g0/0.50
 encapsulation dot1Q 50
 ip address 10.77.0.1 255.255.255.0
 ip helper-address 10.77.255.1
 exit

! Dynamic Routing (EIGRP)
router eigrp 100
 network 10.77.1.0 0.0.0.127
 network 10.77.1.192 0.0.0.63
 network 10.77.0.0 0.0.0.255
 network 10.77.255.0 0.0.0.3
 exit
```

---

## 6. R3 Operations Router (Middle-Bottom Router)

```ios
enable
configure terminal
hostname R3_Ops

! Interface to R1 Core
interface Se0/3/0
 ip address 10.77.255.6 255.255.255.252
 no shutdown
 exit

! Extra Resiliency Interface to R4 (Backup link)
interface Se0/3/1
 ip address 10.77.255.17 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! Main LAN Interface
interface g0/0
 no shutdown
 exit

! Ops VLAN Gateway
interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.77.2.1 255.255.255.224
 ip helper-address 10.77.255.5
 exit

! Admin VLAN Gateway
interface g0/0.40
 encapsulation dot1Q 40
 ip address 10.77.2.33 255.255.255.224
 ip helper-address 10.77.255.5
 exit

! Legacy VLAN Gateway
interface g0/0.80
 encapsulation dot1Q 80
 ip address 10.77.2.113 255.255.255.240
 ip helper-address 10.77.255.5
 exit

! Dynamic Routing (RIPv2)
router rip
 version 2
 no auto-summary
 network 10.77.2.0
 network 10.77.255.4
 network 10.77.255.16
 exit
```

---

## 7. R4 Backup Datacenter Router (Right Router)

```ios
enable
configure terminal
hostname R4_Backup

! Interface to R1 Core
interface Se0/2/0
 ip address 10.77.255.10 255.255.255.252
 no shutdown
 exit

! Extra Resiliency Interface to R3 (Backup link)
interface Se0/3/1
 ip address 10.77.255.18 255.255.255.252
 no shutdown
 exit

! Main LAN Interface
interface g0/0
 no shutdown
 exit

! Backup Site LAN Interface (No VLANs needed, direct)
interface g0/0
 ip address 10.77.1.129 255.255.255.192
 ip helper-address 10.77.255.9
 exit

! Dynamic Routing (OSPF)
router ospf 10
 network 10.77.1.128 0.0.0.63 area 0
 network 10.77.255.8 0.0.0.3 area 0
 network 10.77.255.16 0.0.0.3 area 0
 exit
```
