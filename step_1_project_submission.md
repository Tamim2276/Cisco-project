# Step 1 Project Submission (Final)

## Group Project Title
**E-Sports Tournament Network Design and Simulation using Cisco Packet Tracer**

## Final Project Description (Max 250 Words)
Our project designs and simulates a complete enterprise network for a live E-Sports tournament venue using Cisco Packet Tracer. The network includes the Player Arena, Caster/Broadcast Room, Event Operations Center, Admin/Finance Office, Guest Wi-Fi Zone, DMZ Server Zone, and a Remote Backup Site.

The goal is to build a low-latency, secure, and fault-tolerant tournament infrastructure that can support match operations and live broadcasting without service interruption. To ensure realistic implementation, we will apply VLSM-based IP planning for efficient subnet allocation, VLAN segmentation for traffic isolation, and DHCP for automatic client addressing.

Routing will be implemented using a mixed-protocol design: EIGRP for arena-side segments, OSPF for core/backup connectivity, and RIP for a legacy subnet. Static routing will be used at the WAN edge for ISP/default path control. Route redistribution will be configured at the core to enable full communication across all routing domains.

Security will be enforced through ACL policies, especially to block unauthorized Guest VLAN access to Admin/Finance resources while still allowing approved access to public services (such as web/ticketing servers in the DMZ).

This project demonstrates performance, scalability, segmentation, and security in a real-world CS/networking scenario.
---
## Team Discussion Status
- Team members communicated and finalized the project topic.
- Project title and description approved for Step 1 submission.
- Ready for instructor review and green signal for implementation.