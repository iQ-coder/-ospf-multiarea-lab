# Multi-Area OSPF Lab Writeup

## Objective

Design and implement a multi-area OSPF network simulating a small enterprise with three sites — HQ, Branch 1, and Branch 2 — with MD5 authentication securing all routing communications.

---

## Topology Overview

The network consists of three routers connected in a hub-and-spoke topology. HQ acts as the central hub and is the Area Border Router (ABR) connecting all three OSPF areas. Branch 1 and Branch 2 are spoke sites connected to HQ via serial WAN links.

```
[Branch1 - Area 1]----S0/3/0----[HQ - Area 0]----S0/3/1----[Branch2 - Area 2]
192.168.2.0/24        10.0.0.0/30  192.168.1.0/24  10.0.0.4/30  192.168.3.0/24
```

---

## IP Addressing Scheme

### LAN Networks
| Site     | Subnet          | Gateway       |
|----------|-----------------|---------------|
| HQ       | 192.168.1.0/24  | 192.168.1.1   |
| Branch 1 | 192.168.2.0/24  | 192.168.2.1   |
| Branch 2 | 192.168.3.0/24  | 192.168.3.1   |

### WAN Links (/30 — point-to-point)
| Link             | Network       | HQ Side   | Branch Side |
|------------------|---------------|-----------|-------------|
| HQ ↔ Branch 1   | 10.0.0.0/30   | 10.0.0.1  | 10.0.0.2    |
| HQ ↔ Branch 2   | 10.0.0.4/30   | 10.0.0.5  | 10.0.0.6    |

### Loopback Interfaces (Router IDs)
| Router   | Loopback Address |
|----------|-----------------|
| HQ       | 1.1.1.1/32      |
| Branch 1 | 2.2.2.2/32      |
| Branch 2 | 3.3.3.3/32      |

---

## OSPF Design Decisions

### Why Multi-Area?
Single-area OSPF works fine for small networks but becomes problematic as the network grows. Every router stores the full Link State Database (LSDB) and runs the SPF algorithm against it. With more routers, the LSDB grows larger and SPF calculations become more resource-intensive. Multi-area OSPF solves this by containing topology information within areas — routers only maintain a full LSDB for their own area and receive summarized routes from other areas.

### Area Design
- **Area 0 (Backbone):** Contains HQ and all WAN links. Every OSPF network must have an Area 0 and all other areas must connect to it. The WAN links between HQ and the branches belong to Area 0 because they are part of the backbone infrastructure.
- **Area 1:** Contains Branch 1's LAN (192.168.2.0/24). Isolated from the rest of the topology — only summary routes from other areas are visible here.
- **Area 2:** Contains Branch 2's LAN (192.168.3.0/24). Same isolation principle as Area 1.

### Why HQ is the ABR
An Area Border Router (ABR) is a router with interfaces in more than one OSPF area. HQ has serial interfaces connecting to both Branch 1 (Area 1) and Branch 2 (Area 2) while itself residing in Area 0. This makes HQ responsible for passing summarized routing information between areas. Branch-to-branch traffic always flows through HQ.

### Loopback Interfaces and Router IDs
Each router has a loopback interface configured with a /32 address. OSPF uses this as the Router ID — a unique identifier for each router in the OSPF domain. The reason for using a loopback rather than a physical interface is stability: physical interfaces can go down, which would cause the Router ID to change and destabilize OSPF. A loopback interface is always up as long as the router is powered on, giving OSPF a permanent, reliable identifier.

### /30 Subnets on WAN Links
Point-to-point WAN links only ever have two devices — one router on each end. Using a /30 subnet provides exactly 2 usable host addresses (4 total minus network and broadcast), eliminating any wasted address space. Using a larger subnet like /24 on a WAN link would waste 252 addresses unnecessarily.

---

## Security: OSPF MD5 Authentication

### The Threat
Without authentication, any router that connects to the network and speaks OSPF can form an adjacency with legitimate routers and inject false routing information. This is known as a route injection attack. An attacker could redirect traffic through a malicious device, enabling man-in-the-middle attacks or denial of service.

### The Solution
MD5 authentication requires both ends of an OSPF link to share a secret key. Every OSPF packet includes an MD5 hash of the message combined with the key. The receiving router recomputes the hash using its own copy of the key — if the hashes match, the packet is accepted. If a rogue router doesn't know the key, its packets will be rejected and no adjacency will form.

### Implementation
MD5 authentication was configured on all serial interfaces where OSPF adjacencies form. LAN-facing interfaces (Gig0/0) were not configured since no OSPF neighbors exist on those segments.

```
interface Serial0/3/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 cisco123
```

**Note:** In a production environment, a stronger and less guessable key than "cisco123" should be used.

---

## Verification Commands Used

| Command | Purpose |
|---|---|
| `show ip ospf neighbor` | Verify adjacencies are FULL |
| `show ip ospf interface brief` | Verify interfaces are in correct areas |
| `show ip route` | Verify OSPF routes are learned |
| `show ip ospf database` | Verify LSDB shows multiple areas |
| `ping` | Verify end-to-end connectivity |

---

## Troubleshooting Encountered

During configuration, the OSPF area assignments on the branch routers were incorrect — all interfaces were appearing in Area 0 instead of their respective areas. This was caused by conflicting network statements left over from previous configuration attempts. The fix was to completely remove the OSPF process with `no router ospf 1` and reconfigure it cleanly, which eliminated the duplicate/conflicting entries.

This reinforced an important troubleshooting principle: sometimes a clean restart is more efficient and reliable than trying to patch a broken configuration piece by piece — especially in a lab environment where a brief routing outage is acceptable.

---

## Tools Used
- Cisco Packet Tracer
- Cisco 2911 Routers with HWIC-2T serial modules
- Cisco 2960 Switches
