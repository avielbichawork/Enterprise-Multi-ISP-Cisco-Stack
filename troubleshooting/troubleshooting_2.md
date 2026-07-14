
# 🛠️ Network Troubleshooting & Post-Mortem Guide

This document captures real-world engineering issues encountered during the implementation of the multi-vendor enterprise infrastructure, along with their root-cause analysis and resolutions.

---

## ❌ Case Study 2: Mutual Redistribution Issues & Routing Loops in Redundant L3 Environment (EIGRP & OSPF)

### 📌 Problem Description
In my enterprise core, I designed high availability with redundant connections from my main router (`HQ-ROUTER-1`) to two separate Distribution Layer 3 Switches. 

I implemented mutual redistribution between **OSPF** and **EIGRP Named-Mode** to ensure dynamic routing propagation instead of relying on static routes. 
While OSPF-to-EIGRP redistribution worked immediately after configuring the correct Metric-Type (Type 1), the reverse process—**OSPF to EIGRP**—failed. 

Furthermore, once routing propagation succeeded, simulating a link failure by shutting down the primary interface on `HQ-ROUTER-1` led to potential routing feedback loops across the redundant paths.

---

### 🔍 Detailed Troubleshooting Steps & Root Cause Analysis (RCA)

#### 1. EIGRP Named-Mode Redistribution Metric (FD of Infinity)
When I originally configured redistribution under EIGRP Named-Mode (`address-family ipv4 unicast` -> `topology base`), I noticed the redistributed OSPF routes appeared in the Layer 3 Switch's EIGRP topology table but **did not** transition to the active routing table. 
* **RCA:** Checking the EIGRP topology table revealed that the routes had a **Feasible Distance (FD) of Infinity**. EIGRP requires the manual definition of the 5 vector metric values (Bandwidth, Delay, Reliability, Load, MTU). My initial metric values were too low or mathematically incompatible, causing EIGRP to calculate an infinite composite metric.

#### 2. Suboptimal Routing and Feedback Loops (Redundancy Issue)
With dual-homed connections to both Layer 3 switches, mutual redistribution was happening at multiple boundaries. 
* **RCA:** When I advertised OSPF routes into EIGRP, Switch A propagated them. Switch B then received those EIGRP advertisements and attempted to inject them back into the OSPF domain. This caused suboptimal routing, metric corruption, and intermittent traffic loss during path transitions.

---

### 🛠️ My Engineering Solution & Configuration

To resolve the infinite metric issue and completely prevent routing loops while maintaining full hardware redundancy, I implemented **Route Tagging** using **Route-Maps** on both OSPF and EIGRP distribution points.

#### Step 1: Solving the EIGRP Metric Issue
I re-adjusted the default redistribution seed metric with higher, realistic bandwidth and delay values within the topology base configuration:

```text
! Entering EIGRP Named-Mode configuration
router eigrp HQ-NET
 address-family ipv4 autonomous-system 1
  topology base
   redistribute ospf 1 metric 100000 1000000 255 1 1500
! ==========================================================================
! 1. Route-Maps for EIGRP to OSPF Redistribution
! ==========================================================================
! Block routes that were originally injected from OSPF (Tag 100) to prevent loops
route-map EIGRP-TO-OSPF deny 10
 match tag 100

! Allow remaining EIGRP routes, Tag them with 90, and set OSPF Metric Type 1
route-map EIGRP-TO-OSPF permit 20
 set tag 90
 set metric-type type-1

! ==========================================================================
! 2. Route-Maps for OSPF to EIGRP Redistribution
! ==========================================================================
! Block routes that were originally injected from EIGRP (Tag 90) to prevent loops
route-map OSPF-TO-EIGRP deny 10
 match tag 90

! Allow remaining OSPF routes and Tag them with 110
route-map OSPF-TO-EIGRP permit 20
 set tag 100

! ==========================================================================
! 3. Applying the Route-Maps inside the Routing Processes
! ==========================================================================
router ospf 1
 redistribute eigrp 1 route-map EIGRP-TO-OSPF subnets

router eigrp HQ-NET
 address-family ipv4 autonomous-system 1
  topology base
   redistribute ospf 1 route-map OSPF-TO-EIGRP
```
## ✅ Verification & Failure Simulation
To test the resilience of my configuration, I performed a live failover simulation:

I manually shut down (shutdown) the primary interface on HQ-ROUTER-1 connected to L3 Switch A.

The primary OSPF route was immediately withdrawn from Switch A's table.

Thanks to the backup path, my routes were still correctly advertised through L3 Switch B.

Because the loop-prevention Route-Maps denied tagged routes from being re-injected, no routing loops occurred, and end-to-end branch connectivity remained completely stable throughout the failover.
