# Enterprise Multi-Site Network Architecture & Simulation

## Executive Summary
This project presents an enterprise-grade network architecture designed for **XYZ Research Company**, a high-speed wireless product developer expanding across a multi-building headquarter in Sydney and a remote sales branch in Melbourne.

The network infrastructure is built to support a 100% growth projection over 5 to 10 years, featuring Variable Length Subnet Masking (VLSM), single-area OSPF routing (Area 0), 802.1Q VLAN segmentation, Per-VLAN Spanning Tree load balancing (Rapid PVST+), edge security hardening, and a complete physical layer structured cabling design (MDF/IDF).

* **Academic Module:** 64NETCD – Network Cloud Design
* **Project Team:** Kwan Fai Yew, Leong Yu Feng, Liu An, Desmond Gan

---

## Deliverables & Links

* 🎥 **[Packet Tracer Demo Video Walkthrough (Google Drive)](https://drive.google.com/file/d/1yUXOSApx76jYpxa7mLUBbYGAMzhfQ8wy/view?usp=sharing)**
* 📄 **[Full Technical Architecture Report (PDF)](./DesmondGan_KwanFaiYew_LeongYuFeng_LiuAn_Report.pdf)**
* 📊 **[Executive Project Presentation (PDF)](./DesmondGan_KwanFaiYew_LeongYuFeng_LiuAn_Slides.pdf)**

---

## Network Architecture & Specifications

![Logical Topology Diagram](./Logical%20Topology%20Diagram.png)

### 1. Layer 3 Routing & Traffic Flow (OSPF)
* **Protocol & Area:** OSPF Process ID 50 operating in single Area 0 across all enterprise routers (AR1, AR2, MBR1, RSB1).
* **Router Identifiers & Loopbacks:** Configured unique `/32` loopback interfaces on all routers to serve as deterministic OSPF Router IDs (RSB1: `1.1.1.1`, AR1: `2.1.1.1`, AR2: `2.2.2.2`, MBR1: `3.3.3.3`).
* **Designated Router (DR) Election:** Configured OSPF interface priority (`priority 69`) on edge router AR1's `Gig0/0` interface to enforce AR1 as the Designated Router on the Admin multi-access segment.
* **Control Plane & Default Routing:** Static default route (`0.0.0.0/0`) configured on AR1 pointing to the ISP router and redistributed into OSPF. Passive interfaces enabled on all user-facing LAN edge ports to minimize unnecessary control traffic.

### 2. IP Addressing Scheme & VLSM Allocation
Internal addressing utilizes public Class C address space (`220.0.1.0/24`) subnetted via VLSM to minimize address waste while accommodating 100% capacity growth:

| Network Name | Subnet Address | Subnet Mask | Usable Host Range | Max Hosts | Status / Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Research & Development LAN** | `220.0.1.64` | `/26` (`255.255.255.192`) | `.65` – `.126` | 62 | In Use (Main Building VLAN 10) |
| **Sales & Marketing LAN** | `220.0.1.160` | `/27` (`255.255.255.224`) | `.161` – `.190` | 30 | In Use (Main Building VLAN 20) |
| **Admin LAN** | `220.0.1.192` | `/28` (`255.255.255.240`) | `.193` – `.206` | 14 | In Use (Administration Building) |
| **Remote Sales LAN** | `220.0.1.224` | `/28` (`255.255.255.240`) | `.225` – `.238` | 14 | In Use (Melbourne Branch Office) |
| **Management LAN** | `220.0.1.208` | `/29` (`255.255.255.248`) | `.209` – `.215` | 6 | In Use (Switch SVIs VLAN 99) |
| **Server LAN** | `220.0.1.216` | `/29` (`255.255.255.248`) | `.217` – `.223` | 6 | In Use (Main Building VLAN 30) |
| **Admin WAN Link** | `220.0.1.240` | `/30` (`255.255.255.252`) | `.241` – `.242` | 2 | Point-to-Point (AR1–AR2) |
| **Admin-Main WAN Link** | `220.0.1.248` | `/30` (`255.255.255.252`) | `.249` – `.250` | 2 | Point-to-Point (AR1–MBR1) |
| **Branch-Main WAN Link** | `220.0.1.252` | `/30` (`255.255.255.252`) | `.253` – `.254` | 2 | Point-to-Point (RSB1–MBR1) |
| **ISP WAN Link** | `200.200.100.0` | `/30` (`255.255.255.252`) | `.1` – `.2` | 2 | External Link (AR1 Edge–ISP) |
| *Reserved Subnets* | `220.0.1.0/26`, `220.0.1.128/27`, `220.0.1.244/30` | Mixed | N/A | 94 Total | Reserved for future expansion |

### 3. Layer 2 Switching, Trunking & Spanning Tree (STP)
* **VLAN Segmentation:** Departmental traffic isolated across VLAN 10 (R&D), VLAN 20 (Sales & Marketing), VLAN 30 (Server), VLAN 99 (Management), and VLAN 60 (Unused Ports, administratively shutdown for security).
* **Trunking & VTP:** 802.1Q encapsulation on all inter-switch links with VTP Version 2 domain `MainBuilding` (MBS1: VTP Server, MBS2/MBS3: VTP Clients).
* **Rapid PVST+ Per-VLAN Load Balancing:** Primary and secondary Spanning Tree root bridges assigned per VLAN based on host density distribution across floors to optimize bandwidth and eliminate link bottlenecks:

| VLAN ID | LAN Function | Primary Root Bridge | Secondary Root Bridge | Root Placement Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | Research & Development | **MBS3** (Floor 3) | MBS2 (Floor 2) | Highest host concentration (8 R&D personnel on Floor 3) |
| **VLAN 20** | Sales & Marketing | **MBS2** (Floor 2) | MBS1 (Floor 1) | Highest host concentration (4 Sales personnel on Floor 2) |
| **VLAN 30** | Server Subnet | **MBS1** (Floor 1) | MBS3 (Floor 3) | Direct physical termination of servers on Floor 1 |

* **Port Hardening:** `PortFast` and `BPDU Guard` configured on all access ports to ensure instant host connectivity while protecting against rogue switch attachment.

---

## Physical Layer Infrastructure (L1 Design)

### Main Building Physical Layout Mock-up
![Main Building Mock-up Diagram](./Main%20Building%20Mock-up%20Diagram.png)

### Main Building Cabling & Cable Run Abstract
![Main Building Abstract Diagram](./Main%20Building%20Abstract%20Diagram.png)

* **Structured Cabling Standard:** Category 8.2 Solid-Core Ethernet cables utilized for long-term backbone, Horizontal Cross-Cabling (HCC), and Vertical Cross-Cabling (VCC) between MDF and IDFs. Stranded Cat 8 Ethernet cables used for flexible drop connections from floor outlets to host NICs.
* **Conduit & Installation:** Raised flooring with PVC conduit piping to shield cabling runs across all 3 floors.
* **Bill of Materials (BOM) Summary:** Total physical layer cabling budget calculated at **$49,871.30 SGD** (including 1,928m of Cat 8 cabling, patch panels, wall sockets, and conduit piping). Complete rack equipment budget (Cisco 2901 routers, HWIC-2T modules, Catalyst 2960-24TT switches, patch panels, racks) calculated per MDF/IDF closet.

---

## Verification & Testing Summary

1. **Inter-VLAN Routing & Server Access:** Verified full IP reachability and HTTP service access across all subnets via MBR1 subinterfaces (`g0/0.10`, `g0/0.20`, `g0/0.30`, `g0/0.99`).
2. **Spanning Tree Redundancy Failover:** Simulated single-trunk link failures between MBS1, MBS2, and MBS3; Rapid PVST+ successfully reconverged traffic flow with sub-second failover times without packet loss.
3. **Centralized TFTP Configuration Backup:** Device running configurations (`NETCD-AR1-config`, `NETCD-MBS1-config`, etc.) automatically backed up to the dedicated TFTP Server on `220.0.1.219`.
