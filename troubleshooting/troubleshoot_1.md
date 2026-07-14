# 🛠️ Network Troubleshooting & Post-Mortem Guide

This document captures real-world engineering issues encountered during the implementation of the multi-vendor enterprise infrastructure, along with their root-cause analysis and resolutions.

---

## ❌ Case Study 1: Broken NAT Translations During ISP Failover (Dual-Homed Edge)

### 📌 Problem Description
The enterprise network uses dual internet service providers:
* **Primary Link (ISP-2):** Connected via `200.200.200.0/30` with IP SLA and Object Tracking.
* **Secondary Link (ISP-1):** Connected via `100.100.100.0/30` with a higher Administrative Distance (Backup).

During a simulated failure of the primary link (ISP-2), the IP SLA successfully triggered, and the static default route switched to the secondary provider (ISP-1). Internal users could ping the local ISP-1 interfaces, but **all outbound internet traffic was completely dropped.**

---

### 🔍 Root Cause Analysis (RCA)
Although the routing table updated dynamically, Cisco's default **NAT Translation Table** retained active translations pointing to the failed ISP-2 public IP subnet. 

When internal hosts attempted to access the internet via ISP-1, the router continued to translate their private source IPs to the public IP of **ISP-2** (which was down) and forwarded them over the ISP-1 link. ISP-1 dropped these packets due to anti-spoofing filters, causing a total outage for transiting users.

---

### 🛠️ Solution & Resolution Configuration

To fix this, I implemented **Route-Map-based Dynamic NAT** tied to the physical egress interfaces, coupled with strict NAT translation timers to force rapid session clearing.

```text
! 1. Define local subnet access list
access-list 1 permit 192.168.0.0 0.0.255.255
access-list 1 permit 10.0.0.0 0.0.255.255

! 2. Create Route-Maps matching traffic to specific outbound interfaces
route-map RM-NAT-ISP2 permit 10
 match ip address 1
 match interface Ethernet0/1  <-- Primary ISP-2 Interface

route-map RM-NAT-ISP1 permit 10
 match ip address 1
 match interface Ethernet0/0  <-- Secondary ISP-1 Interface

! 3. Apply Interface-specific NAT overload bindings
ip nat inside source route-map RM-NAT-ISP2 interface Ethernet0/1 overload
ip nat inside source route-map RM-NAT-ISP1 interface Ethernet0/0 overload

