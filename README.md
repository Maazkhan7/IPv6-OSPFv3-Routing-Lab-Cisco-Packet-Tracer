# 🌐 IPv6 OSPFv3 Routing Lab — Cisco Packet Tracer

A hands-on Cisco Packet Tracer lab demonstrating **IPv6 dynamic routing with OSPFv3** between two routers, each serving its own LAN. The lab walks through configuration, neighbor adjacency, routing table verification, and the OSPFv3 link-state database — and highlights one of the biggest quality-of-life differences between how IPv4 and IPv6 routing protocols are enabled.

> 💡 **Note:** This lab uses **OSPFv3**, Cisco's IPv6 link-state routing protocol (confirmed by the `ipv6 router ospf 1` and `show ipv6 ospf` output below). The same "beauty" principle described here — interface-based protocol enablement — applies identically to **IPv6 RIP (RIPng)**, which is why it's called out explicitly in this README.

---

## 📌 Table of Contents
- [Topology](#-topology)
- [Objectives](#-objectives)
- [Addressing Scheme](#-addressing-scheme)
- [Configuration](#-configuration)
- [Verification](#-verification)
- [The Beauty of IPv6 Routing: Interface-Based Enablement](#-the-beauty-of-ipv6-routing-interface-based-enablement)
- [Key Learnings](#-key-learnings)
- [Outcomes](#-outcomes)
- [Tools Used](#-tools-used)

---

## 🗺️ Topology

```
        Loopback 10.1.1.1/24                          Loopback 20.1.1.1/24
               |                                              |
           [ R1 - 1941 ]  --- Gig0/1 <2001::1/64 ↔ 2001::2/64> Gig0/1 ---  [ R2 - 1941 ]
               |                                              |
           Gig0/0                                          Gig0/0
        a:a:a:a::1/64                                  b:b:b:b::1/64
               |                                              |
          [ Switch0 ]                                    [ Switch1 ]
          /    |    \                                    /    |    \
      PC1    PC2    PC3                              PC1    PC2    PC3
   a:a:a:a::2  ::3   ::4                          b:b:b:b::2  ::3   ::4
```

Two Cisco 1941 routers (**R1**, **R2**), each with:
- A **Loopback0** interface used as the OSPF Router ID
- A **LAN segment** (Gig0/0) connected to a switch with 3 PCs
- A **point-to-point link** (Gig0/1) directly connecting R1 ↔ R2

---

## 🎯 Objectives

- Configure IPv6 addressing across two routers and their LANs
- Enable **OSPFv3** as the dynamic routing protocol between R1 and R2
- Verify neighbor adjacency, routing tables, and the LSA database
- Demonstrate how IPv6 routing protocols are enabled **per-interface**, unlike IPv4's network-statement approach

---

## 📋 Addressing Scheme

| Device | Interface | IPv6 Address | Role |
|---|---|---|---|
| R1 | Loopback0 | 10.1.1.1/24 | Router ID source |
| R1 | Gig0/1 | 2001::1/64 | R1 ↔ R2 link |
| R1 | Gig0/0 | a:a:a:a::1/64 | LAN gateway |
| R2 | Loopback0 | 20.1.1.1/24 | Router ID source |
| R2 | Gig0/1 | 2001::2/64 | R1 ↔ R2 link |
| R2 | Gig0/0 | b:b:b:b::1/64 | LAN gateway |
| R1 LAN PCs | — | a:a:a:a::2/64 – a:a:a:a::4/64 | End hosts |
| R2 LAN PCs | — | b:b:b:b::2/64 – b:b:b:b::4/64 | End hosts |

---

## ⚙️ Configuration

**Router 1 (R1):**
```
Router(config)# hostname R1
R1(config)# interface loopback 0
R1(config-if)# ip address 10.1.1.1 255.255.255.255

R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ipv6 address 2001:0000:0000:0000:0000:0000:0000:0001/64
R1(config-if)# ipv6 ospf 1 area 0

R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ipv6 address 000a:000a:000a:000a:000a:000a:000a:0001/64
R1(config-if)# ipv6 ospf 1 area 0

R1(config)# ipv6 router ospf 1
```

**Router 2 (R2):**
```
R2(config)# interface gigabitEthernet 0/1
R2(config-if)# ipv6 address 2001:0000:0000:0000:0000:0000:0000:0002/64
R2(config-if)# ipv6 ospf 1 area 0

R2(config)# interface loopback 0
R2(config-if)# ip address 20.1.1.1 255.255.255.255

R2(config)# ipv6 router ospf 1

R2(config)# interface gigabitEthernet 0/0
R2(config-if)# ipv6 address 000b:000b:000b:000b:000b:000b:000b:0001/64
R2(config-if)# ipv6 ospf 1 area 0
```

> ⚠️ Notice: **no `network` command was ever used.** OSPFv3 is turned on individually on each interface with `ipv6 ospf 1 area 0`. The `ipv6 router ospf 1` process just creates the process — the interfaces decide what gets advertised.

---

## ✅ Verification

**1. OSPFv3 protocol summary — `show ipv6 protocols`**
```
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0)
    GigabitEthernet0/1
    GigabitEthernet0/0
  Redistribution: None
```

**2. Neighbor adjacency — `show ipv6 ospf neighbor`**
```
R1:  Neighbor ID 20.1.1.1   State FULL/BDR   Interface GigabitEthernet0/1
R2:  Neighbor ID 10.1.1.1   State FULL/DR    Interface GigabitEthernet0/1
```
Full adjacency confirmed on both ends — R1 is DR, R2 is BDR.

**3. Routing table — `show ipv6 route`**
```
R1:
O  B:B:B:B::/64 [110/2] via FE80::20C:85FF:FEDC:CD02, GigabitEthernet0/1
C  A:A:A:A::/64 [0/0]   directly connected
C  2001::/64    [0/0]   directly connected

R2:
O  A:A:A:A::/64 [110/2] via FE80::204:9AFF:FE7D:2A02, GigabitEthernet0/1
C  B:B:B:B::/64 [0/0]   directly connected
C  2001::/64    [0/0]   directly connected
```
Each router learned the **remote LAN** (`O` = OSPF) with a link-local next-hop — proof that OSPFv3 is fully operational end to end.

**4. LSA Database — `show ipv6 ospf database`**
```
Router Link States (Area 0): 10.1.1.1, 20.1.1.1
Net Link States (Area 0):     10.1.1.1 (DR)
Link (Type-8) States (Area 0): Gi0/0, Gi0/1 entries for both routers
Intra Area Prefix Link States: prefixes for both LANs and the transit link
```

**5. Interface-level OSPFv3 detail — `show ipv6 ospf interface`**
```
GigabitEthernet0/1: State DR, DR ID 10.1.1.1, BDR ID 20.1.1.1, Cost 1
GigabitEthernet0/0: State DR, No backup DR on this network, Cost 1
```

---

## 💡 The Beauty of IPv6 Routing: Interface-Based Enablement

This is the core takeaway of the lab, and it applies to **both OSPFv3 and IPv6 RIP (RIPng)**:

### In IPv4:
Routing protocols like OSPF and RIP are enabled by **advertising a network** under the routing process:
```
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
```
👉 If the IP address on that interface later changes (re-IP, re-subnet, migration, etc.), the `network` statement becomes **stale or wrong**, and you must go back into the routing process and **reconfigure the network statement** to match the new address/subnet. Miss it, and the interface silently stops being advertised.

### In IPv6:
OSPFv3 (and IPv6 RIP) are enabled **directly on the interface itself**:
```
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
```
or, for IPv6 RIP:
```
interface GigabitEthernet0/0
 ipv6 rip PROCESS_NAME enable
```
👉 There is **no separate network statement to maintain**. The protocol is bound to the interface, not to a specific address. If the IPv6 address on that interface changes tomorrow, OSPFv3/RIPng **automatically advertises whatever address is configured now** — nothing to go back and edit at the process level.

**In short:**
| | IPv4 (OSPF/RIP) | IPv6 (OSPFv3/RIPng) |
|---|---|---|
| Enablement | `network` statement under process | `ipv6 ospf`/`ipv6 rip` under interface |
| On IP change | Must edit the network statement | Nothing to change — interface config carries it |
| Risk | Easy to forget/misconfigure after re-IP | Self-maintaining |

This lab captures that exact behavior — every interface running OSPFv3 was enabled with a single interface-level command, and the routing table adapted automatically without ever touching a `network` line.

---

## 🧠 Key Learnings

- OSPFv3 uses **Router IDs** derived from loopback addresses, not IPv6 addresses
- Neighbor discovery and Hello/Dead timers work identically to OSPFv2, just over IPv6/link-local addressing
- DR/BDR election still applies on broadcast (Ethernet) segments in OSPFv3
- IPv6 next-hops in the routing table are **link-local (FE80::)** addresses, not global addresses
- Interface-based protocol enablement removes an entire class of IPv4-style misconfiguration

---

## 🏁 Outcomes

- ✅ Full OSPFv3 neighbor adjacency (FULL/DR ↔ FULL/BDR) established between R1 and R2
- ✅ End-to-end IPv6 reachability between both LANs (`a:a:a:a::/64` ↔ `b:b:b:b::/64`)
- ✅ Correct OSPFv3 database population (Router, Network, Link, Intra-Area-Prefix LSAs)
- ✅ Demonstrated and documented the interface-based enablement advantage of IPv6 routing protocols over IPv4

---

## 🛠️ Tools Used

- Cisco Packet Tracer
- Cisco IOS (1941 Router, 2950-24 Switch)
- OSPFv3 (IPv6 link-state routing)

---

### 📎 Related
Part of an ongoing series of **Cisco/GNS3 networking lab projects** — building a public portfolio of practical IPv4/IPv6 routing, switching, and troubleshooting exercises.
