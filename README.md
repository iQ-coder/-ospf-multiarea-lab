# Multi-Area OSPF Network Lab

A simulated enterprise network built in Cisco Packet Tracer demonstrating multi-area OSPF routing with MD5 authentication.

## Overview

This lab simulates a company with three sites — HQ, Branch 1, and Branch 2 — connected via WAN serial links. OSPF is used as the dynamic routing protocol, organized into three areas with HQ acting as the Area Border Router (ABR). MD5 authentication is configured on all OSPF links to protect against route injection attacks.

## Topology

```
[Branch1]----WAN----[HQ]----WAN----[Branch2]
 Area 1           Area 0           Area 2
192.168.2.0/24  192.168.1.0/24  192.168.3.0/24
```

## Key Concepts Demonstrated

- Multi-area OSPF design (Area 0, Area 1, Area 2)
- Area Border Router (ABR) role and inter-area routing
- Loopback interfaces for stable OSPF Router IDs
- /30 subnets for point-to-point WAN links
- OSPF MD5 authentication to prevent route injection attacks
- LSDB verification and routing table analysis

## IP Addressing

| Device   | Interface  | IP Address      | Area   |
|----------|------------|-----------------|--------|
| HQ       | Loopback0  | 1.1.1.1/32      | Area 0 |
| HQ       | Gig0/0     | 192.168.1.1/24  | Area 0 |
| HQ       | S0/3/0     | 10.0.0.1/30     | Area 0 |
| HQ       | S0/3/1     | 10.0.0.5/30     | Area 0 |
| Branch1  | Loopback0  | 2.2.2.2/32      | Area 1 |
| Branch1  | Gig0/0     | 192.168.2.1/24  | Area 1 |
| Branch1  | S0/3/0     | 10.0.0.2/30     | Area 0 |
| Branch2  | Loopback0  | 3.3.3.3/32      | Area 2 |
| Branch2  | Gig0/0     | 192.168.3.1/24  | Area 2 |
| Branch2  | S0/3/1     | 10.0.0.6/30     | Area 0 |

## Repository Structure

```
ospf-multiarea-lab/
├── README.md
├── configs/
│   ├── HQ.txt          # HQ router running config
│   ├── Branch1.txt     # Branch 1 router running config
│   └── Branch2.txt     # Branch 2 router running config
└── docs/
    └── lab-writeup.md  # Detailed design and security writeup
```

## Security Considerations

OSPF MD5 authentication is configured on all serial interfaces to prevent unauthorized routers from forming adjacencies and injecting false routing information. See the full writeup in `docs/lab-writeup.md` for a detailed explanation of the threat model and mitigation.

## Tools Used

- Cisco Packet Tracer
- Cisco 2911 Routers (with HWIC-2T serial modules)
- Cisco 2960 Switches

## Part of a Series

This is the third project in a networking portfolio series:
1. Small Office Network with DHCP
2. VLAN Segmentation with Subinterfaces and Telnet
3. **Multi-Area OSPF with MD5 Authentication** ← you are here
