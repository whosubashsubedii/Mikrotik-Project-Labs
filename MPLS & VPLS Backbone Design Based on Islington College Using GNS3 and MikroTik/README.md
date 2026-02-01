# Islington College MPLS & VPLS Backbone Design Using GNS3 and MikroTik

## Table of Contents
1. [Project Overview](#project-overview)
2. [Network Architecture](#network-architecture)
3. [IP Addressing Plan](#ip-addressing-plan)
4. [VLANs and Subnets](#vlans-and-subnets)
5. [Core Network Design](#core-network-design)
6. [MPLS & LDP Configuration](#mpls--ldp-configuration)
7. [VPLS Configuration](#vpls-configuration)
8. [Centralized DHCP Server](#centralized-dhcp-server)
9. [Redundancy and Load Balancing](#redundancy-and-load-balancing)
10. [End-to-End Verification](#end-to-end-verification)
11. [Network Topology Diagrams](#network-topology-diagrams)
12. [Troubleshooting Guide](#troubleshooting-guide)
13. [Security and Access Control](#security-and-access-control)
14. [Files in Repository](#files-in-repository)
15. [Conclusion and Future Enhancements](#conclusion-and-future-enhancements)

---

## Project Overview
This project implements a **multi-block MPLS backbone network** for **Islington College** using **MikroTik routers in GNS3**. The design provides **VPLS-based Layer-2 connectivity** across all blocks, centralized DHCP services, and redundancy with ECMP/load balancing. 

The **main block (LONDON_BLOCK)** connects to the Internet, while all other blocks connect directly and via additional mesh links for high availability.

---

## Network Architecture
- **Core Routers:** All block routers act as MPLS core routers.
- **IGP:** OSPF is deployed across all routers.
- **MPLS:** Enabled with LDP for label distribution.
- **ECMP / Load Balancing:** Implemented across core and mesh links.
- **VPLS:** Used to extend VLANs across all blocks.
- **Centralized DHCP Server:** Provides IPs for end devices; no DHCP running on routers.

---

## IP Addressing Plan

### Router Loopback Addresses (Public /32)
| Block | Loopback IP       |
|-------|-----------------|
| LONDON_BLOCK | 160.30.132.1 |
| UK_BLOCK     | 160.30.132.11 |
| NEPAL_BLOCK  | 160.30.132.12 |
| HIMAL_BLOCK  | 160.30.132.13 |
| BRIT_BLOCK   | 160.30.132.14 |
| SKILL_BLOCK  | 160.30.132.15 |
| ALUMNI_BLOCK | 160.30.132.16 |
| KUMARI_BLOCK | 160.30.132.17 |

### Point-to-Point Links from LONDON_BLOCK (/30 Private IPs)
| Link                  | LONDON | Remote |
|-----------------------|--------|--------|
| LONDON → UK           | 10.0.0.1 | 10.0.0.2 |
| LONDON → NEPAL        | 10.0.0.5 | 10.0.0.6 |
| LONDON → HIMAL        | 10.0.0.9 | 10.0.0.10 |
| LONDON → BRIT         | 10.0.0.13 | 10.0.0.14 |
| LONDON → SKILL        | 10.0.0.17 | 10.0.0.18 |
| LONDON → KUMARI       | 10.0.0.21 | 10.0.0.22 |
| LONDON → ALUMNI       | 10.0.0.25 | 10.0.0.26 |

### Mesh Links (/30 Private IPs)
| Link                  | Node1 IP | Node2 IP |
|-----------------------|----------|----------|
| UK → NEPAL            | 10.0.0.29 | 10.0.0.30 |
| UK → HIMAL            | 10.0.0.34 | 10.0.0.33 |
| NEPAL → BRIT          | 10.0.0.37 | 10.0.0.38 |
| HIMAL → SKILL         | 10.0.0.42 | 10.0.0.41 |
| BRIT → KUMARI         | 10.0.0.46 | 10.0.0.45 |
| SKILL → ALUMNI        | 10.0.0.50 | 10.0.0.49 |
| ALUMNI → KUMARI       | 10.0.0.53 | 10.0.0.54 |

---

## VLANs and Subnets (Centralized DHCP)
| VLAN ID | Name      | Subnet           | Capacity |
|---------|-----------|----------------|----------|
| 100     | STUDENT   | 172.16.0.0/19   | 5,000 IPs |
| 200     | TEACHER   | 172.16.32.0/19  | 1,000 IPs |
| 300     | STAFF     | 172.16.36.0/21  | 2,000 IPs |

---

## Core Network Design
- All routers configured with **loopback IPs**.
- Core interfaces configured with **/30 IP addresses**.
- **OSPF** configured with loopbacks advertised and ECMP paths enabled.
- **MPLS enabled on all core and mesh links**.
- **VPLS deployed point-to-point** for VLAN extension across blocks.
- **MPLS MTU adjusted** to prevent fragmentation in VPLS.

---

## MPLS & LDP Configuration
- MPLS enabled on all core interfaces.
- **LDP sessions established** between all directly connected routers.
- Loopback IPs used for **LDP router IDs**.
- ECMP applied for load balancing on multiple paths.

---

## VPLS Configuration
- Point-to-point VPLS tunnels configured between blocks.
- VLANs extended over MPLS backbone using VPLS.
- VPLS transport ensures **Layer-2 connectivity** for student, teacher, and staff VLANs.

---

## Centralized DHCP Server
- Located at **LONDON_BLOCK**.
- Provides IPs for all VLANs across all blocks.
- Routers configured to **forward DHCP requests** via IP helper (no DHCP on routers themselves).

---

## Redundancy and Load Balancing
- Mesh links between blocks provide **redundant paths**.
- **ECMP in OSPF/MPLS** ensures traffic is load-balanced across multiple paths.
- VPLS ensures VLAN continuity in case of link failure.

---

## End-to-End Verification
- Verify connectivity to **all loopbacks**.
- Check **MPLS LSPs** are established.
- Test **VPLS VLAN extension** by pinging devices in the same VLAN across blocks.
- Validate **DHCP IP allocation** to end devices.

---

## Network Topology Diagrams
- `Design.jpeg.jpg` – Full network topology
- `ISLINGTON VPLS & MPLS.jpg` – MPLS & VPLS overview
- Detailed GNS3 project files included in:
  - `Islington College MPLS Project/`
  - `MPLS & VPLS Backbone Design based Islington College Using GNS3 and Mi.docx`

---

## Troubleshooting Guide
- **OSPF issues:** Verify neighbors and link costs.
- **MPLS/LDP problems:** Check LDP sessions and label bindings.
- **VPLS VLAN bridging errors:** Verify VPLS interfaces and VLAN tags.
- **DHCP failures:** Ensure IP helper configured on routers and server reachable.
- **ECMP/Load Balancing verification:** Check routing table and traffic distribution.

---

## Security and Access Control
- **Management Access:** SSH/Winbox to all routers.
- **ACLs/Firewall Rules:** Applied to prevent unauthorized access on core interfaces.

---

## Files in Repository
- GNS3 project files and router configurations.
- Network diagrams (`.jpg`) and documentation (`.docx`).
- This `README.md` for project reference.

---

## Conclusion and Future Enhancements
- Fully functional **MPLS/VPLS backbone network** for Islington College.
- Centralized DHCP ensures uniform IP management.
- Future improvements:
  - IPv6 deployment for all blocks.
  - QoS implementation for bandwidth-sensitive applications.
  - Automated network monitoring and alerting.


---

## Author
**Subash Subedi**  
GitHub: [@whosubashsubedii](https://github.com/whosubashsubedii)  
Email: project@subashsubedi0.com.np
