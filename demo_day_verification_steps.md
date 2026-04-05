# Demo Day Verification Steps (Complete Checklist)

Use this file live during presentation to verify every required point.

## 0. Quick Rule

For all ping tests:

1. First ping may timeout once (ARP learning) and still be okay.
2. Pass condition is stable replies after that.

---

## 1. Verify DHCP Leases (Players, Guests, Admin, Backup)

## 1.1 Player PC (VLAN 10)

On Player PC Command Prompt:

```bat
:: Show client IP, mask, and gateway (VLAN 10 lease)
ipconfig
:: VLAN 10 gateway (Players network 10.77.1.0/25)
ping 10.77.1.1
```

Pass if:

1. IP is in `10.77.1.0/25`
2. Gateway is `10.77.1.1`
3. Ping to gateway succeeds

## 1.2 Guest PC (VLAN 50)

On Guest PC:

```bat
:: Show client IP, mask, and gateway (VLAN 50 lease)
ipconfig
:: VLAN 50 gateway (Guests network 10.77.0.0/24)
ping 10.77.0.1
```

Pass if:

1. IP is in `10.77.0.0/24`
2. Gateway is `10.77.0.1`
3. Ping to gateway succeeds

## 1.3 Admin PC (VLAN 40)

On Admin PC:

```bat
:: Show client IP, mask, and gateway (VLAN 40 lease)
ipconfig
:: VLAN 40 gateway (Admin network 10.77.2.32/27)
ping 10.77.2.33
```

Pass if:

1. IP is in `10.77.2.32/27`
2. Gateway is `10.77.2.33`
3. Ping to gateway succeeds

## 1.4 Backup Endpoint (Server5, VLAN 90)

On Server5:

```bat
:: Show client IP, mask, and gateway (VLAN 90 lease)
ipconfig
:: VLAN 90 gateway (Backup users network 10.77.1.128/26)
ping 10.77.1.129
```

Pass if:

1. IP is in `10.77.1.128/26`
2. Gateway is `10.77.1.129`
3. Ping succeeds

---

## 2. Verify Inter-VLAN Routing (Authorized Users)

Use Player or Admin PC:

```bat
:: DMZ Web server (VLAN 60 / 10.77.2.96/28)
ping 10.77.2.98
:: DMZ Ticket server (VLAN 60 / 10.77.2.96/28)
ping 10.77.2.99
:: DMZ Stream/NTP server (VLAN 60 / 10.77.2.96/28)
ping 10.77.2.100
```

Pass if:

1. Pings to DMZ servers succeed
2. Cross-VLAN communication works

---

## 3. Verify ACL Blocks Guest -> Admin

On Guest PC:

```bat
:: Admin gateway (VLAN 40 / 10.77.2.32/27) - should be blocked
ping 10.77.2.33
:: Operations gateway (VLAN 30 / 10.77.2.0/27) - should be blocked
ping 10.77.2.1
```

Pass if:

1. Both fail (timeouts)

Reason:

1. ACL `GUEST_ISOLATION` deny rules block Guest access to protected internal VLANs.

---

## 4. Verify Guest Can Access Allowed DMZ Services

On Guest PC:

```bat
:: DMZ Web server (VLAN 60 / 10.77.2.96/28) - allowed target
ping 10.77.2.98
```

Pass if:

1. Ping succeeds

Then on R2 to prove ACL activity:

```ios
enable
show access-lists GUEST_ISOLATION
```

Pass if:

1. Deny and permit lines show increasing `matches` counters

---

## 5. Check Routing Tables on All Routers

## 5.1 R1_Core

```ios
enable
show ip route
show ip protocols
```

Pass if:

1. `D`, `O`, and `R` routes are present
2. Default route points to `10.77.255.14`

## 5.2 R2_Arena

```ios
enable
show ip eigrp neighbors
show ip route eigrp
```

Pass if:

1. EIGRP neighbor exists
2. EIGRP routes exist

## 5.3 R3_Operations

```ios
enable
show ip route rip
show ip rip database
```

Pass if:

1. RIP routes visible

## 5.4 R4_Backup

```ios
enable
show ip ospf neighbor
show ip route ospf
```

Pass if:

1. OSPF neighbor state is FULL
2. OSPF routes visible

## 5.5 R5_ISP

```ios
enable
show ip route
! R1 edge IP on R1-R5 WAN link (10.77.255.12/30)
ping 10.77.255.13
```

Pass if:

1. Static return route to `10.77.0.0/16` exists
2. Ping to R1 edge succeeds

---

## 6. Simulate WAN Failure and Verify Alternate Path

Suggested link to test down:

1. Bring down `R1 <-> R3` serial path temporarily

On R1:

```ios
enable
configure terminal
interface se0/3/1
 shutdown
end
```

Then test from Ops/Legacy side:

```bat
:: DMZ Web server - used to verify reroute via backup path
ping 10.77.2.98
```

Pass if:

1. Traffic still works after convergence (may drop briefly, then recover)
2. Alternate path through `R3 -> R4 -> R1` is used

Restore link after test:

```ios
enable
configure terminal
interface se0/3/1
 no shutdown
end
```

---

## 7. Save Configs on All Routers and Switches

Run on each router and switch:

```ios
enable
copy running-config startup-config
```

At prompt:

`Destination filename [startup-config]?`

Action:

1. Press Enter

Pass if:

1. You see `Building configuration...` and `[OK]`

---

## 8. Proof Screenshots to Capture

1. DHCP `ipconfig` from Player, Guest, Admin, Server5
2. Guest blocked ping to Admin
3. Guest allowed ping to DMZ server
4. `show access-lists GUEST_ISOLATION` with hit counters
5. `show ip route` on R1
6. `show ip ospf neighbor` on R4
7. `show ip eigrp neighbors` on R2
8. WAN failure test before/after ping results
9. `[OK]` from save command

If all above pass, your project verification is complete.
