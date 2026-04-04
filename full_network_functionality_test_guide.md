# Full Network Functionality Test Guide (All Devices)

Use this file to test the complete Packet Tracer project from edge to core.

## 1. Test Order (Recommended)

1. Physical/interface health
2. Switch VLAN/trunk checks
3. Router routing checks
4. DHCP checks on clients
5. Inter-VLAN and inter-site ping checks
6. ACL behavior checks
7. Server role checks
8. Save configurations

---

## 2. Router Tests

## 2.1 R1_Core (Core + DHCP + Redistribution)

Run on R1:

```ios
enable
show ip interface brief
show ip route
show ip protocols
show ip ospf neighbor
show ip eigrp neighbors
show ip rip database
show ip dhcp binding
show running-config | include ip route 0.0.0.0
```

Expected:

1. Active interfaces are up/up.
2. Routes from EIGRP, OSPF, RIP are visible.
3. DHCP bindings exist for branch users.
4. Default route to ISP exists.

## 2.2 R2_Arena (EIGRP + ACL)

Run on R2:

```ios
enable
show ip interface brief
show ip eigrp neighbors
show ip route eigrp
show access-lists GUEST_ISOLATION
show ip interface g0/0.50
show running-config | section interface GigabitEthernet0/0.50
```

Expected:

1. Se0/3/0 and G0/0 subinterfaces are up.
2. EIGRP neighbor with R1 is up.
3. Guest ACL is applied inbound on G0/0.50.
4. ACL hit counters increase during tests.

## 2.3 R3_Operations (RIP)

Run on R3:

```ios
enable
show ip interface brief
show ip protocols
show ip route rip
show ip rip database
```

Expected:

1. Se0/3/0 and Se0/3/1 are up.
2. RIP v2 active with learned routes.

## 2.4 R4_Backup (OSPF + Backup LAN)

Run on R4:

```ios
enable
show ip interface brief
show ip ospf neighbor
show ip route ospf
ping 10.77.255.9
```

Expected:

1. Se0/3/0, Se0/3/1, G0/0 are up/up.
2. OSPF neighbor and OSPF routes are present.
3. Ping to R1 backup link succeeds.

## 2.5 R5_ISP (Static Return)

Run on R5:

```ios
enable
show ip interface brief
show ip route
show running-config | include ip route 10.77.0.0
ping 10.77.255.13
```

Expected:

1. Se0/3/0 is up/up.
2. Static return route to 10.77.0.0/16 exists.
3. Ping to R1 succeeds.

---

## 3. Switch Tests

## 3.1 S1_Core

```ios
enable
show vlan brief
show interfaces trunk
```

Check:

1. VLAN 60 and 70 exist.
2. G0/1 trunk is up.
3. F0/2-4 in VLAN 60, F0/5 in VLAN 70.

## 3.2 S2_Arena

```ios
enable
show vlan brief
show interfaces trunk
```

Check:

1. VLAN 10, 20, 50 exist.
2. G0/1 trunk is up.
3. F0/2 in VLAN 10, F0/3 in VLAN 20, F0/4 in VLAN 50.

## 3.3 S3_Operations

```ios
enable
show vlan brief
show interfaces trunk
```

Check:

1. VLAN 30, 40, 80 exist.
2. G0/1 trunk is up.
3. F0/2 in VLAN 30, F0/3 in VLAN 40, F0/4 in VLAN 80.

## 3.4 S4_Backup

```ios
enable
show vlan brief
```

Check:

1. VLAN 90 exists.
2. Uplink and backup endpoint ports are in VLAN 90.

---

## 4. PC and Endpoint Tests

Open Command Prompt on each endpoint.

## 4.1 Arena PCs

On Player/Caster/Guest PCs:

```bat
ipconfig
ping 10.77.1.1
ping 10.77.2.98
```

Expected:

1. DHCP addresses are assigned in correct subnet.
2. Default gateway reachable.
3. DMZ host reachable if policy allows.

## 4.2 Operations PCs

On Ops/Admin/Legacy PCs:

```bat
ipconfig
ping 10.77.2.1
ping 10.77.2.33
ping 10.77.2.98
```

Expected:

1. Correct DHCP subnet and gateway.
2. Cross-VLAN reachability works.

## 4.3 Backup Endpoint (Server5)

```bat
ipconfig
ping 10.77.1.129
ping 10.77.255.9
ping 10.77.2.98
```

Expected:

1. IP in 10.77.1.128/26 range.
2. R4 gateway reachable.
3. Inter-site ping works.

---

## 5. Server Tests

## 5.1 Static Server IP Validation

On SRV1/SRV2/SRV3/SRV4 Desktop > IP Configuration, verify:

1. SRV1: 10.77.2.98/28, GW 10.77.2.97
2. SRV2: 10.77.2.99/28, GW 10.77.2.97
3. SRV3: 10.77.2.100/28, GW 10.77.2.97
4. SRV4: 10.77.2.66/27, GW 10.77.2.65

## 5.2 Server Reachability

From one branch PC:

```bat
ping 10.77.2.98
ping 10.77.2.99
ping 10.77.2.100
ping 10.77.2.66
```

Expected:

1. DMZ service servers respond according to policy.
2. Management server reachability depends on ACL rules.

---

## 6. ACL Validation (Required)

Run from Guest PC (VLAN 50):

```bat
ping 10.77.2.33
ping 10.77.2.1
ping 10.77.2.64
ping 10.77.2.113
ping 10.77.2.98
ping 10.77.255.14
```

Expected:

1. Admin/Operations/Management/Legacy targets are blocked.
2. Allowed DMZ target and upstream permitted traffic pass.

Then on R2:

```ios
show access-lists GUEST_ISOLATION
```

Expected:

1. Deny and permit counters increase, proving ACL is active.

---

## 7. End-to-End Routing Proof

Run from one endpoint in each site:

1. Local gateway ping
2. Ping one endpoint/server in another site
3. Ping DMZ server

Optional traceroute proof:

```bat
tracert 10.77.2.98
```

Use this to show multi-hop path through core.

---

## 8. Save Config on All Devices

Run on each router and switch:

```ios
enable
copy running-config startup-config
```

Press Enter when asked:

`Destination filename [startup-config]?`

---

## 9. Demo-Ready Proof Checklist

Capture screenshots of:

1. `show ip route` on R1
2. `show ip protocols` on R1
3. `show access-lists GUEST_ISOLATION` on R2
4. DHCP lease (`ipconfig`) from Arena/Operations/Backup endpoint
5. Backup endpoint inter-site ping success
6. R5 ping to R1 edge IP

If all checks pass, the project is fully validated.

---

## 10. Sample Expected Output (Quick Reference)

Use these patterns to compare your screen output.

## 10.1 `show ip interface brief` (router)

Success pattern:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     ...             YES manual up                    up
Serial0/...            ...             YES manual up                    up
```

Important check:

1. Active links must show `up` / `up`.

## 10.2 `show ip route` (R1)

Success pattern:

```text
Gateway of last resort is 10.77.255.14 to network 0.0.0.0
...
D    10.77.x.x ...
O    10.77.x.x ...
R    10.77.x.x ...
```

Important check:

1. You should see `D`, `O`, and `R` routes together.
2. Default route to ISP should be present.

## 10.3 `show ip eigrp neighbors` (R1/R2)

Success pattern:

```text
EIGRP-IPv4 Neighbors for AS(100)
H   Address        Interface    Hold  Uptime ...
0   10.77.255.1/2  Se0/3/0      ...
```

Important check:

1. Neighbor row exists (not empty).

## 10.4 `show ip ospf neighbor` (R1/R4)

Success pattern:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
...             ...   FULL/...        ...         10.77.255.x     Se0/...
```

Important check:

1. State contains `FULL`.

## 10.5 `show ip route rip` (R1/R3)

Success pattern:

```text
R    10.77.x.x/... [120/..] via 10.77.255.6, ...
```

Important check:

1. One or more `R` routes appear.

## 10.6 `show vlan brief` (switch)

Success pattern:

```text
VLAN Name                             Status    Ports
10   Players                          active    Fa0/2
20   Casters                          active    Fa0/3
...  ...
```

Important check:

1. Required VLAN IDs exist and access ports map correctly.

## 10.7 `show interfaces trunk` (switch)

Success pattern:

```text
Port      Mode         Encapsulation  Status        Native vlan
Gi0/1     on           802.1q         trunking      1
```

Important check:

1. Uplink `Gi0/1` shows `trunking`.

## 10.8 `ipconfig` (PC/Server)

Success pattern:

```text
IPv4 Address...........: 10.77.x.x
Subnet Mask............: 255.255.x.x
Default Gateway........: 10.77.x.x
```

Important check:

1. Not `169.254.x.x`.
2. Gateway matches VLAN router IP.

## 10.9 `ping` tests

Success pattern:

```text
Reply from 10.77.x.x: bytes=32 time<1ms TTL=...
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Notes:

1. First ping may timeout once because of ARP learning.
2. After that, replies should succeed.

## 10.10 `show access-lists GUEST_ISOLATION` (R2)

Success pattern:

```text
Extended IP access list GUEST_ISOLATION
	deny ip 10.77.0.0 0.0.0.255 10.77.2.32 0.0.0.31 (x matches)
	...
	permit ip 10.77.0.0 0.0.0.255 any (y matches)
```

Important check:

1. `matches` counters increase when you run guest tests.

## 10.11 `copy running-config startup-config`

Success pattern:

```text
Destination filename [startup-config]?   (press Enter)
Building configuration...
[OK]
```

Important check:

1. `[OK]` confirms configuration is saved.

---

## 11. Full Ping Matrix (Every PC and Server)

Use this section to test each endpoint one by one.

Output meaning:

1. `Reply from ...` = reached.
2. `Request timed out` for first packet only = often normal ARP learning.
3. `Request timed out` for all packets = blocked path or configuration issue.

### 11.1 Player_PC (VLAN 10)

Run:

```bat
ping 10.77.1.1
ping 10.77.2.98
ping 10.77.1.129
```

Expected output:

1. Gateway ping succeeds (Reply, 0% loss).
2. DMZ ping succeeds (Reply, maybe first packet timeout only).
3. Backup site ping succeeds.

If failed, common reason:

1. Wrong VLAN/port on S2.
2. Missing R2 subinterface or helper/routing issue.

### 11.2 Caster_PC (VLAN 20)

Run:

```bat
ping 10.77.1.193
ping 10.77.2.99
ping 10.77.2.1
```

Expected output:

1. Gateway ping succeeds.
2. Ticket server ping succeeds.
3. Ops gateway ping succeeds (inter-site routing proof).

If failed, common reason:

1. Port not in VLAN 20.
2. R2 G0/0.20 down/misconfigured.

### 11.3 Guest_PC (VLAN 50, ACL enforced)

Run:

```bat
ping 10.77.0.1
ping 10.77.2.33
ping 10.77.2.1
ping 10.77.2.98
ping 10.77.255.14
```

Expected output:

1. `10.77.0.1` succeeds (local gateway).
2. `10.77.2.33` fails (Admin blocked by ACL).
3. `10.77.2.1` fails (Operations blocked by ACL).
4. `10.77.2.98` succeeds (allowed DMZ path).
5. `10.77.255.14` succeeds (upstream allowed by ACL permit any).

Why it failed or reached:

1. Failed to Admin/Ops is expected due ACL deny lines.
2. Reached DMZ/upstream is expected due ACL permit lines.

### 11.4 Ops_PC (VLAN 30)

Run:

```bat
ping 10.77.2.1
ping 10.77.2.98
ping 10.77.1.129
```

Expected output:

1. Gateway ping succeeds.
2. DMZ ping succeeds.
3. Backup site ping succeeds.

If failed, common reason:

1. VLAN 30 switch port mismatch.
2. RIP/redistribution path not learned.

### 11.5 Admin_PC (VLAN 40)

Run:

```bat
ping 10.77.2.33
ping 10.77.2.66
ping 10.77.2.99
```

Expected output:

1. Gateway ping succeeds.
2. Syslog/management server ping succeeds.
3. Ticket server ping succeeds.

If failed, common reason:

1. Wrong VLAN 40 access port.
2. R3 G0/0.40 config issue.

### 11.6 Legacy_PC (VLAN 80)

Run:

```bat
ping 10.77.2.113
ping 10.77.2.100
ping 10.77.1.1
```

Expected output:

1. Gateway ping succeeds.
2. Stream/NTP server ping succeeds.
3. Arena gateway ping succeeds (cross-domain path).

If failed, common reason:

1. VLAN 80 port assignment wrong.
2. RIP route or redistribution missing.

### 11.7 Server5 (Backup endpoint, VLAN 90)

Run:

```bat
ipconfig
ping 10.77.1.129
ping 10.77.255.9
ping 10.77.2.98
```

Expected output:

1. `ipconfig` shows 10.77.1.130+ and GW 10.77.1.129.
2. Gateway ping succeeds.
3. Ping to R1 backup link and DMZ succeeds.

If failed, common reason:

1. DHCP relay from R4 to R1 broken.
2. OSPF adjacency/path down.

### 11.8 SRV1_Web (10.77.2.98)

Run:

```bat
ping 10.77.2.97
ping 10.77.1.1
```

Expected output:

1. R1 DMZ gateway ping succeeds.
2. Arena gateway ping succeeds.

If failed, common reason:

1. Wrong static IP/gateway on SRV1.
2. S1 VLAN 60 port mismatch.

### 11.9 SRV2_Ticket (10.77.2.99)

Run:

```bat
ping 10.77.2.97
ping 10.77.2.33
```

Expected output:

1. Gateway ping succeeds.
2. Admin gateway ping succeeds.

If failed, common reason:

1. VLAN 60 assignment wrong.
2. R1 subinterface issue.

### 11.10 SRV3_Stream (10.77.2.100)

Run:

```bat
ping 10.77.2.97
ping 10.77.2.1
```

Expected output:

1. Gateway ping succeeds.
2. Operations gateway ping succeeds.

If failed, common reason:

1. DMZ VLAN issue.
2. RIP/redistribution path issue.

### 11.11 SRV4_Syslog (10.77.2.66)

Run:

```bat
ping 10.77.2.65
ping 10.77.2.98
```

Expected output:

1. Management gateway ping succeeds.
2. DMZ server ping succeeds.

If failed, common reason:

1. VLAN 70 access port mismatch.
2. R1 G0/0.70 issue.

## 11.12 Why Ping Fails vs Why Ping Reaches (Quick Explanation)

Ping fails when:

1. ACL intentionally blocks that traffic.
2. Wrong IP/subnet/gateway on endpoint.
3. Wrong VLAN on switch port.
4. Interface is down/down or administratively down.
5. Route to destination is missing.

Ping reaches when:

1. Endpoint IP/mask/gateway are correct.
2. Switch access/trunk VLANs are correct.
3. Router interfaces and subinterfaces are up.
4. Dynamic/static routing provides a valid path.
5. ACL has a permit for that traffic.
