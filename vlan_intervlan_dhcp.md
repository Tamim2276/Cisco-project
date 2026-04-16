# VLAN, Inter-VLAN Routing, and DHCP in the E-Sports Tournament Network

## VLANs (Virtual Local Area Networks)

VLANs are used in this project to logically segment the network into different broadcast domains, even though devices may be connected to the same physical switch. Each group of users or devices (Players, Casters, Guests, Admins, Operations, Legacy, Backup, Management, DMZ Servers) is assigned to its own VLAN, providing both security and traffic isolation.

- Example VLANs:
  - VLAN 10: Players (10.77.1.0/25)
  - VLAN 20: Casters (10.77.1.192/26)
  - VLAN 50: Guests (10.77.0.0/24)
  - VLAN 60: DMZ Servers (10.77.2.96/28)
  - VLAN 70: Management (10.77.2.64/27)
  - VLAN 90: Backup Users (10.77.1.128/26)

VLANs are configured on all switches, and trunk ports connect switches to routers to carry multiple VLANs over a single physical link.

## Inter-VLAN Routing

By default, devices in different VLANs cannot communicate. Inter-VLAN routing is required to enable communication between VLANs. In this project, inter-VLAN routing is implemented using router-on-a-stick:

- Each router (e.g., R2_Arena, R3_Operations) has subinterfaces on its main LAN interface (e.g., G0/0.10, G0/0.20, G0/0.50), each configured for a different VLAN and assigned the gateway IP for that VLAN.
- The router performs routing between VLANs, allowing authorized traffic to flow according to the security policies and ACLs.
- Example: R2_Arena provides gateways for VLANs 10 (Players), 20 (Casters), and 50 (Guests) using subinterfaces G0/0.10, G0/0.20, and G0/0.50.

## DHCP (Dynamic Host Configuration Protocol)

DHCP is used to automatically assign IP addresses and network settings to end devices in each VLAN. In this project:

- R1_Core acts as the centralized DHCP server, with a separate DHCP pool for each user VLAN (Players, Casters, Guests, etc.).
- Routers at each site (R2, R3, R4) use the `ip helper-address` command on their VLAN subinterfaces to relay DHCP requests from clients to the DHCP server on R1.
- This ensures that devices in all VLANs receive the correct IP configuration, default gateway, and DNS settings automatically.

## Summary

- VLANs provide logical segmentation and security.
- Inter-VLAN routing (router-on-a-stick) enables communication between VLANs as needed.
- Centralized DHCP with relay ensures all devices are configured correctly and efficiently.

This design supports secure, scalable, and manageable networking for the E-Sports tournament environment.
