---

## 📊 Addressing & VLAN Plan

### Campus LAN (Headquarters)
The HQ infrastructure utilizes EIGRP Named-Mode across core and distribution layers, with strict boundary segregation via tailored VLAN allocation:

| VLAN ID | Subnet | Gateway Placement | Description / Purpose |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | `10.10.10.0/24` | `10.10.10.254` (VRRP) | HR |
| **VLAN 20** | `10.10.20.0/24` | `10.10.20.254` (VRRP) | IT |
| **VLAN 30** | `10.10.30.0/24` | `10.10.30.254` (VRRP) | Management |
| **VLAN 40** | `10.10.40.0/24` | `10.10.40.254` (VRRP) | Sales |
| **VLAN 50** | `10.10.50.0/24` | `10.10.50.254` (VRRP) | Guest Network |
| **VLAN 99** | `10.10.99.0/24` | N/A | Native VLAN (Unused / Hardened) |

### Remote Branch LAN
The Remote Branch relies on an OSPF Area 1 Stub architecture to maintain a lightweight routing table:

| VLAN ID | Subnet | Gateway Placement | Description / Purpose |
| :--- | :--- | :--- | :--- |
| **VLAN 110** | `10.10.110.0/24` | `10.10.110.254` (VRRP) | HR |
| **VLAN 120** | `10.10.120.0/24` | `10.10.120.254` (VRRP) | IT |
| **VLAN 130** | `10.10.130.0/24` | `10.10.130.254` (VRRP) | Management |
| **VLAN 140** | `10.10.140.0/24` | `10.10.140.254` (VRRP) | Sales |
| **VLAN 150** | `10.10.150.0/24` | `10.10.150.254` (VRRP) | Guest Network |
| **VLAN 199** | `10.10.199.0/24` | N/A | Native VLAN (Unused / Hardened) |

### WAN & Overlay Point-to-Point Interconnects

| Segment / Link | IP Subnet | Allocation | Routing Domain |
| :--- | :--- | :--- | :--- |
| **HQ Link to ISP-2 (Primary)** | `200.200.200.0/30` | HQ: `.1`, ISP-2: `.2` | eBGP (AS 500 ↔ AS 200) |
| **HQ Link to ISP-1 (Backup)** | `100.100.100.0/30` | HQ: `.1`, ISP-1: `.2` | eBGP (AS 500 ↔ AS 100) |
| **Branch Link to ISP-2 (Primary)** | `200.200.200.4/30` | Branch: `.5`, ISP-2: `.6` | eBGP (AS 500 ↔ AS 200) |
| **Branch Link to ISP-1 (Backup)** | `100.100.100.4/30` | Branch: `.5`, ISP-1: `.6` | eBGP (AS 500 ↔ AS 100) |
| **DMVPN Tunnel 1 Overlay** | `172.16.1.0/24` | Hub: `.1`, Spoke: `.2` | OSPF Area 0 (Primary Path) |
| **DMVPN Tunnel 2 Overlay** | `172.16.2.0/24` | Hub: `.1`, Spoke: `.2` | OSPF Area 0 (Backup Path) |

---

## 🔄 WAN Path Selection & Failover Matrix

To ensure maximum uptime and intelligent path selection, the architecture evaluates link health continuously using Cisco IP SLA and Object Tracking. Traffic steering is strictly deterministic:

### Path Selection Logic
1. **Primary State:** All corporate data and DMVPN overlay traffic is forwarded via **ISP-2** (Tunnel 1). This is enforced via eBGP Local Preference (`150` towards ISP-2, `100` towards ISP-1) on the underlay, and OSPF cost adaptation on the overlay.
2. **Failure Detection:** The HQ Edge router generates continuous ICMP Echo probes (`IP SLA 1`) targeting the ISP-2 next-hop address (`200.200.200.2`). Probes are monitored via `track 1`.
3. **Failover Execution:** If ISP-2 experiences failure or severe packet loss, the IP SLA state shifts to *Down*. The tracking object instantly triggers a routing switchover:
   * The static/BGP default route pointing to ISP-2 is pulled from the RIB.
   * Internal OSPF traffic dynamically converges to **Tunnel 2 (ISP-1)**.
4. **Failback Behavior:** Once ISP-2 stabilizes for a continuous duration (configured via a hysteresis delay), the IP SLA object returns to *Up* and restores the primary default route, gracefully shifting production traffic back to the high-speed ISP-2 link.

| Scenario | Primary Path (ISP-2) | Backup Path (ISP-1) | Expected Behavior / Recovery Mechanism |
| :--- | :--- | :--- | :--- |
| **Normal Operations** | **Active (Online)** | Standby (Online) | All traffic routes over ISP-2. DMVPN Tunnel 1 forms the primary overlay path. |
| **ISP-2 Link Failure** | ❌ Dead | **Active (Online)** | IP SLA 1 fails ➔ Object Track 1 triggers ➔ ISP-2 default route pulled ➔ OSPF overlay shifts traffic completely to Tunnel 2 (ISP-1). |
| **ISP-2 Recovery** | **Active (Online)** | Standby (Online) | Link stabilization timer expires ➔ IP SLA returns to UP ➔ Traffic seamlessly transitions back to ISP-2. |
