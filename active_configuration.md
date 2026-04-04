# Active Project Commands

Here are exactly the commands you have used so far, in the order you are building your network.

## 1. R1 Core Router (WAN Interfaces)

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
```

## 2. S1 Core Switch

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

## 3. Server Static IPs

Click each server -> **Desktop** -> **IP Configuration**, then set them statically:

- **Game Server (SRV1 on F0/2 - VLAN 60):**
  - IP Address: `10.77.2.98`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **Database Server (SRV2 on F0/3 - VLAN 60):**
  - IP Address: `10.77.2.99`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **NTP Server (SRV3 on F0/4 - VLAN 60):**
  - IP Address: `10.77.2.100`
  - Subnet Mask: `255.255.255.240`
  - Default Gateway: `10.77.2.97`

- **Syslog Server (SRV4 on F0/5 - VLAN 70):**
  - IP Address: `10.77.2.66`
  - Subnet Mask: `255.255.255.224`
  - Default Gateway: `10.77.2.65`

## 4. R1 Core Router (LAN Subinterfaces for S1)

```ios
enable
configure terminal

! --- LAN Interface (Router-on-a-Stick for DMZ and Mgmt) ---
! First, turn on the physical port
interface g0/0
 no shutdown
 exit

! Create the subinterface for VLAN 60 (DMZ Servers)
interface g0/0.60
 encapsulation dot1Q 60
 ip address 10.77.2.97 255.255.255.240
 exit

! Create the subinterface for VLAN 70 (Management / Syslog)
interface g0/0.70
 encapsulation dot1Q 70
 ip address 10.77.2.65 255.255.255.224
 exit
```

## 5. R1 Core Router (DHCP Server Pools)

```ios
enable
configure terminal

! --- Exclude Gateway IPs from being assigned to PCs ---
ip dhcp excluded-address 10.77.1.1
ip dhcp excluded-address 10.77.1.193
ip dhcp excluded-address 10.77.0.1
ip dhcp excluded-address 10.77.2.1
ip dhcp excluded-address 10.77.2.33
ip dhcp excluded-address 10.77.2.113
ip dhcp excluded-address 10.77.1.129

! --- DHCP Pools for Arena, Ops, and Backup Branches ---

ip dhcp pool VLAN10_PLAYERS
 network 10.77.1.0 255.255.255.128
 default-router 10.77.1.1
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN20_CASTERS
 network 10.77.1.192 255.255.255.192
 default-router 10.77.1.193
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN50_GUESTS
 network 10.77.0.0 255.255.255.0
 default-router 10.77.0.1
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN30_OPS
 network 10.77.2.0 255.255.255.224
 default-router 10.77.2.1
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN40_ADMIN
 network 10.77.2.32 255.255.255.224
 default-router 10.77.2.33
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN80_LEGACY
 network 10.77.2.112 255.255.255.240
 default-router 10.77.2.113
 dns-server 8.8.8.8
 exit

ip dhcp pool VLAN90_BACKUP
 network 10.77.1.128 255.255.255.192
 default-router 10.77.1.129
 dns-server 8.8.8.8
 exit
```

## 6. R1 Core Router (Routing Protocols)

```ios
enable
configure terminal

! --- Static Route to ISP (R5) ---
! All unknown internet traffic goes out via Se0/2/1
ip route 0.0.0.0 0.0.0.0 10.77.255.14

! --- OSPF (Core, Servers, and Backup Site) ---
router ospf 10
 ! Advertise DMZ Server VLAN
 network 10.77.2.96 0.0.0.15 area 0
 ! Advertise Management VLAN
 network 10.77.2.64 0.0.0.31 area 0
 ! Advertise WAN link to Backup Router
 network 10.77.255.8 0.0.0.3 area 0
 ! Bring in routes from EIGRP and RIP
 redistribute eigrp 100 subnets
 redistribute rip subnets
 ! Send the default route (internet) to OSPF neighbors
 default-information originate
 exit

! --- EIGRP (For Arena Site) ---
router eigrp 100
 ! Advertise WAN link to Arena Router
 network 10.77.255.0 0.0.0.3
 ! Bring in routes from OSPF and RIP
 redistribute ospf 10 metric 10000 100 255 1 1500
 redistribute rip metric 10000 100 255 1 1500
 exit

! --- RIP (For Operations Site) ---
router rip
 version 2
 no auto-summary
 ! Advertise WAN link to Ops Router
 network 10.0.0.0
 ! Note: In RIP, we use the classful boundary (10.0.0.0), RIPv2 will handle the classless routing
 redistribute ospf 10 metric 2
 redistribute eigrp 100 metric 2
 default-information originate
 exit
```

## 7. R2 Arena Router (WAN, LAN, & Routing)

```ios
enable
configure terminal
hostname R2_Arena

! --- WAN Interface back to R1 Core ---
interface Se0/3/0
 ip address 10.77.255.2 255.255.255.252
 no shutdown
 exit

! --- LAN Interface (Router-on-a-Stick for S2) ---
interface g0/0
 no shutdown
 exit

! Players VLAN (VLAN 10)
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.77.1.1 255.255.255.128
 ! Important: Tell clients to ask R1 for DHCP
 ip helper-address 10.77.255.1
 exit

! Casters VLAN (VLAN 20)
interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.77.1.193 255.255.255.192
 ip helper-address 10.77.255.1
 exit

! Guests VLAN (VLAN 50)
interface g0/0.50
 encapsulation dot1Q 50
 ip address 10.77.0.1 255.255.255.0
 ip helper-address 10.77.255.1
 exit

! --- Dynamic Routing (EIGRP) ---
router eigrp 100
 network 10.77.1.0 0.0.0.127
 network 10.77.1.192 0.0.0.63
 network 10.77.0.0 0.0.0.255
 network 10.77.255.0 0.0.0.3
 exit
```

## 8. S2 Arena Switch (VLANs & Access Ports)

```ios
enable
configure terminal
hostname S2_Arena

! Create the VLANs
vlan 10
 name Players
vlan 20
 name Casters
vlan 50
 name Guests
 exit

! Configure the Trunk port pointing to the Router (R2)
interface g0/1
 switchport mode trunk
 exit

! Configure Access port for Players (VLAN 10)
interface f0/2
 switchport mode access
 switchport access vlan 10
 exit

! Configure Access port for Casters (VLAN 20)
interface f0/3
 switchport mode access
 switchport access vlan 20
 exit

! Configure Access port for Guests (VLAN 50)
interface f0/4
 switchport mode access
 switchport access vlan 50
 exit
```

## 9. R3 Operations Router (WAN, LAN, & Routing)

```ios
enable
configure terminal
hostname R3_Operations

! --- WAN Interface back to R1 Core (on Se0/3/0) ---
interface Se0/3/0
 ip address 10.77.255.6 255.255.255.252
 no shutdown
 exit

! --- WAN Interface to R4 Backup (on Se0/3/1) ---
interface Se0/3/1
 ip address 10.77.255.17 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! --- LAN Interface (Router-on-a-Stick for S3) ---
interface g0/0
 no shutdown
 exit

! Ops VLAN (VLAN 30)
interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.77.2.1 255.255.255.224
 ! Tell clients to ask R1 for DHCP
 ip helper-address 10.77.255.5
 exit

! Admin VLAN (VLAN 40)
interface g0/0.40
 encapsulation dot1Q 40
 ip address 10.77.2.33 255.255.255.224
 ip helper-address 10.77.255.5
 exit

! Legacy VLAN (VLAN 80)
interface g0/0.80
 encapsulation dot1Q 80
 ip address 10.77.2.113 255.255.255.240
 ip helper-address 10.77.255.5
 exit

! --- Dynamic Routing (RIP Version 2) ---
router rip
 version 2
 no auto-summary
 ! RIP uses the classful network boundary to find internal networks
 network 10.0.0.0
 exit
```

## 10. S3 Operations Switch (VLANs & Access Ports)

```ios
enable
configure terminal
hostname S3_Operations

! Create the VLANs
vlan 30
 name Ops_Staff
vlan 40
 name Admin_Staff
vlan 80
 name Legacy_Devices
 exit

! Configure the Trunk port pointing to the Router (R3)
interface g0/1
 switchport mode trunk
 exit

! Configure Access port for Ops (VLAN 30)
interface f0/2
 switchport mode access
 switchport access vlan 30
 exit

! Configure Access port for Admin (VLAN 40)
interface f0/3
 switchport mode access
 switchport access vlan 40
 exit

! Configure Access port for Legacy (VLAN 80)
interface f0/4
 switchport mode access
 switchport access vlan 80
 exit
```

## 11. R4 Backup Datacenter Router (WAN, LAN, & Routing)

```ios
enable
configure terminal
hostname R4_Backup

! --- WAN Interface back to R1 Core (on Se0/3/0) ---
interface Se0/3/0
 ip address 10.77.255.10 255.255.255.252
 no shutdown
 exit

! --- WAN Interface to R3 Ops (on Se0/3/1) ---
! (This builds in the triangle resiliency link)
interface Se0/3/1
 ip address 10.77.255.18 255.255.255.252
 no shutdown
 exit

! --- LAN Interface (No VLANs, Flat Network) ---
interface g0/0
 ip address 10.77.1.129 255.255.255.192
 ! Direct DHCP requests to R1 Core
 ip helper-address 10.77.255.9
 no shutdown
 exit

! --- Dynamic Routing (OSPF) ---
router ospf 10
 ! Advertise Backup Site LAN
 network 10.77.1.128 0.0.0.63 area 0
 ! Advertise link to R1 Core
 network 10.77.255.8 0.0.0.3 area 0
 ! Advertise link to R3 Ops
 network 10.77.255.16 0.0.0.3 area 0
 exit
```

## 12. R5 ISP Router (WAN & Return Route)

```ios
enable
configure terminal
hostname R5_ISP

! --- WAN Interface to R1 Core ---
interface Se0/3/0
 ip address 10.77.255.14 255.255.255.252
 clock rate 64000
 no shutdown
 exit

! --- Static Return Route to Enterprise Network ---
! So ISP knows how to send replies back to all your internal subnets
ip route 10.77.0.0 255.255.0.0 10.77.255.13
```

## 13. Final Verification Checks

Run these from key devices to confirm the full network is working:

```ios
! On R1_Core
show ip route
show ip protocols

! On R4_Backup
show ip interface brief
ping 10.77.255.9

! On Server5 (Desktop > Command Prompt)
ipconfig
ping 10.77.1.129
ping 10.77.255.9
ping 10.77.2.98
```

## 14. Basic Security Hardening

```ios
! Apply on every router and switch
enable
configure terminal

enable secret Cisco123!
service password-encryption

banner motd #
Authorized access only. Disconnect immediately if you are not allowed.
#

line console 0
 password Console123!
 login
 exit

line vty 0 4
 password Vty123!
 login
 exit

end
```

## 15. ACL Implementation (Guest Isolation on R2)

```ios
! Apply on R2_Arena
enable
configure terminal

! Block Guest VLAN from reaching sensitive internal VLANs
ip access-list extended GUEST_ISOLATION
 deny ip 10.77.0.0 0.0.0.255 10.77.2.32 0.0.0.31
 deny ip 10.77.0.0 0.0.0.255 10.77.2.0 0.0.0.31
 deny ip 10.77.0.0 0.0.0.255 10.77.2.64 0.0.0.31
 deny ip 10.77.0.0 0.0.0.255 10.77.2.112 0.0.0.15

 ! Allow Guest users to reach DMZ services
 permit tcp 10.77.0.0 0.0.0.255 host 10.77.2.98 eq 80
 permit tcp 10.77.0.0 0.0.0.255 host 10.77.2.99 eq 443
 permit icmp 10.77.0.0 0.0.0.255 10.77.2.96 0.0.0.15

 ! Allow all other traffic (including internet-bound traffic)
 permit ip 10.77.0.0 0.0.0.255 any
 exit

! Apply ACL inbound on Guest VLAN subinterface
interface g0/0.50
 ip access-group GUEST_ISOLATION in
 exit

end
```

### ACL Test Checklist

From a Guest PC:

1. `ping 10.77.2.33` -> should FAIL (Admin VLAN blocked)
2. `ping 10.77.2.1` -> should FAIL (Operations VLAN blocked)
3. `ping 10.77.2.98` -> should PASS (DMZ allowed)
4. `ping 10.77.255.14` -> should PASS (upstream reachability)

On R2:

1. `show access-lists GUEST_ISOLATION`
2. `show ip interface g0/0.50`
3. `show running-config | section interface GigabitEthernet0/0.50`

```ios
enable
copy running-config startup-config
```
