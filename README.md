# Multi-ISP Cisco Enterprise Infrastructure Stack

Cisco Enterprise Network Architecture fully simulated and validated in EVE-NG.

The objective of this project was to design and validate an enterprise network that prioritizes high availability, resilient WAN connectivity, secure campus access, and controlled interoperability between multiple routing domains following common enterprise network design practices such as routing domain separation, redundant WAN connectivity, and controlled route redistribution.

---

## 🌐 Enterprise Network Overview

---

## 🗺️ Network Topology

![Enterprise Network Topology](Topology/network_design.png)

*Note: The source EVE-NG `.unl` topology file is available in the root directory of this repository for lab replication.*

### 📊 Topology Color Legend
| Color Boundary / Element | Description & Architectural Role |
| :--- | :--- |
| **Purple Box (Left)** | **HQ Campus LAN:** EIGRP Named-Mode Core/Distribution layer with VRRP and LACP. |
| **Green Box (Right)** | **Remote Branch LAN:** OSPF Area 1 Stub area domain for local campus routing. |
| **Pink Box (Center)** | **Secure WAN Overlay:** Dual-Tunnel DMVPN (Phase 3) running GRE over IPsec. |
| **Top Infrastructure** | **Public WAN Underlay:** eBGP peering connections between HQ/Branch Edge and ISPs. |

---

### 🚀 Architectural Highlights

A high-impact, scannable overview of the implemented architecture:

* **Topology Core:** Corporate Headquarters + Redundant Remote Branch
* **WAN Edge:** Dual ISP Connectivity with dynamic primary/backup failover
* **Secure VPN Overlays:** Dual-Tunnel Dynamic Multipoint VPN (DMVPN Phase 3)
* **WAN Routing Underlay:** eBGP peering with public service providers (AS-100 & AS-200)
* **WAN Routing Overlay:** OSPF over DMVPN tunnels
* **Campus LAN Core:** EIGRP-based Named-Mode campus Core/Distribution layer (AS-500)
* **Edge Interoperability:** Mutual Redistribution with Route-Maps & Route-Tag loop prevention
* **First Hop Redundancy:** VRRP for local gateway resilience at the HQ Distribution layer
* **Layer 2 Infrastructure:** Rapid Per-VLAN Spanning Tree (RPVSTP+) & LACP EtherChannel
* **Path Monitoring:** IP SLA + Object Tracking for continuous link health monitoring
* **Edge Security:** Policy-Based PAT (NAT Overload) using advanced Route-Maps
* **LAN Hardening:** Layer 2 protection against rogue DHCP servers, ARP spoofing, and unauthorized endpoint access

---

## 🎯 Design Goals

The core engineering objectives behind the technological choices made in this project:

* ✔ **High Availability:** Reduction of single points of failure across critical LAN and WAN paths.
* ✔ **Dynamic WAN Failover:** Automated path shifting during primary ISP degradation without manual intervention.
* ✔ **Fast Convergence:** Controlled failover and deterministic routing convergence during topology changes.
* ✔ **Secure Campus LAN:** Layer 2 protection using DHCP Snooping, Dynamic ARP Inspection (DAI), and Port Security.
* ✔ **Enterprise Scalability:** Designed to support future branch expansion using a scalable DMVPN architecture.
* ✔ **Routing Plane Isolation:** Strict separation maintained between WAN service provider boundaries (eBGP) and internal routing domains.
* ✔ **Future NGFW Readiness:** The topology was designed with dedicated transit links to allow future insertion of a Next-Generation Firewall (NGFW) without redesigning the Layer 3 architecture.

---

## 🛠️ Detailed Technical Documentation

To keep the overview concise, the complete low-level engineering specifications, subnet allocations, and failover design parameters have been segmented into dedicated architectural documents:

* 📊 **[Addressing & VLAN Plan](docs/addressing-and-routing.md)** – Comprehensive subnet tables for HQ LAN, Remote Branch, and WAN point-to-point links.
* 🔄 **[WAN Path Selection & Failover Matrix](docs/addressing-and-routing.md#-wan-path-selection--failover-matrix)** – Detailed logic covering IP SLA configuration, Object Tracking, and eBGP/OSPF convergence behaviors during link failures.

---

## 🔍 Infrastructure Verification & Validation

To review the live state validation of the network control plane, DMVPN overlay, security associations, and high-availability tracking, please refer to the complete verification document:

[![Verification Guide](https://img.shields.io/badge/Documentation-Verification_Guide-blue?style=for-the-badge&logo=read-the-docs&logoColor=white)](verification/verification.md)

---

## 🛠️ Network Troubleshooting (Post-Mortems)

During the implementation of this multi-vendor infrastructure, I documented key technical issues, their root causes, and my step-by-step resolution processes:

* ❌ **[Case Study 1: Broken NAT Translations During ISP Failover](troubleshooting/troubleshoot_1.md)**  
  *Resolving persistent NAT translation states on dual-homed edge paths during a simulated ISP blackout.*
* ❌ **[Case Study 2: Mutual Redistribution & Routing Loops (EIGRP & OSPF)](troubleshooting/troubleshooting_2.md)**  
  *Fixing "Infinity Metric" values in EIGRP Named-Mode and implementing Route-Maps with Route-Tagging to eliminate feedback loops.*

---

## 🎓 Lessons Learned

Building and testing this architecture in EVE-NG provided crucial production-level insights:
1. **NAT State Persistence:** Learned that routing table convergence via IP SLA isn't enough for true WAN failover; NAT translation tables must be explicitly managed or timed out to prevent traffic drops over the backup link.
2. **Redistribution Risks:** Gained deep practical knowledge on why route tagging is non-negotiable in multi-point mutual redistribution to prevent routing loops and metric corruption.
3. **Underlay/Overlay Separation:** Validated how separating public infrastructure routing (eBGP) from enterprise overlay transit (OSPF/DMVPN) isolates internal topology data and simplifies routing troubleshooting.
