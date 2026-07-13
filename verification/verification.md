# 🔍 Infrastructure Verification Guide

This document outlines the systematic verification commands used to validate control plane stability, overlay infrastructure health, and deterministic data plane forwarding behavior across the multi-vendor environment.

---

## 🌐 1. Dynamic Routing Verification

### Underlay BGP Peer Validation
To verify that the Edge Routers successfully establish eBGP peerings with the service providers (ISP-1 and ISP-2) and receive correct routing advertisements:

```bash
HQ-ROUTER#show ip bgp su
BGP router identifier 200.200.200.1, local AS number 500
BGP table version is 10, main routing table version 10
7 network entries using 980 bytes of memory
12 path entries using 960 bytes of memory
7/4 BGP path/bestpath attribute entries using 1008 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3044 total bytes of memory
BGP activity 7/0 prefixes, 12/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.100.100.2   4          100     296     296       10    0    0 04:23:32        5
200.200.200.2   4          200     296     297       10    0    0 04:23:42        5 '''
קטע קוד
HQ-ROUTER#show ip bgp neighbors 200.200.200.2 advertised-routes 
BGP table version is 10, local router ID is 200.200.200.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  8.8.8.8/32       200.200.200.2                      200      0 200 1000 i
 *>  100.100.100.0/30 0.0.0.0                  0         32768 i
 *>  100.100.100.4/30 100.100.100.2            0             0 100 i
 *>  200.200.200.0/30 0.0.0.0                  0         32768 i
 *>  200.200.200.4/30 200.200.200.2            0    200      0 200 i
 *>  209.165.100.0/30 200.200.200.2            0    200      0 200 i
 *>  209.165.100.4/30 200.200.200.2                      200      0 200 1000 i

Total number of prefixes 7 
Enterprise Core EIGRP Validation (Named-Mode)
Validating EIGRP interior routing convergence across the campus core and distribution layers:

קטע קוד
HQ-ROUTER#show ip eigrp neighbors 
EIGRP-IPv4 VR(NAME-1) Address-Family Neighbors for AS(1)
H   Address                 Interface           Hold Uptime   SRTT   RTO  Q  Seq
                                                (sec)         (ms)       Cnt Num
1   10.0.0.5                Et0/0                    12 04:25:21    8   100  0  34
0   10.0.0.1                Et0/1                    12 04:25:21    6   100  0  33
קטע קוד
HQ-ROUTER#show ip eigrp topology 
EIGRP-IPv4 VR(NAME-1) Topology Table for AS(1)/ID(200.200.200.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 192.168.10.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 172.16.0.2/32, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.110.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.140.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 172.16.100.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 172.16.100.2/32, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.30.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 172.16.0.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 10.0.0.0/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/1
P 192.168.40.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 192.168.130.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.150.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 0.0.0.0/0, 1 successors, FD is 131072000
        via Rstatic (131072000/0)
P 192.168.50.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 10.0.0.4/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/0
P 192.168.120.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.20.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
🔒 2. DMVPN Phase-3 & IPsec Validation
Crypto ISAKMP & IPsec SA Verification
Ensuring that Phase 1 and Phase 2 security associations are fully negotiated and encrypting transit overlay tunnels:

קטע קוד
HQ-ROUTER#show crypto isakmp sa 
IPv4 Crypto ISAKMP SA
dst             src             state        conn-id status
200.200.200.1   200.200.200.5   QM_IDLE           1002 ACTIVE
100.100.100.1   100.100.100.5   QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA
NHRP (Next Hop Resolution Protocol) Mapping
Verifying the dynamic NHRP registration matrix on the Hub for all connected Spoke routers:

קטע קוד
HQ-ROUTER#show ip nhrp brief 
   Target             Via             NBMA            Mode   Intfc   Claimed 
     172.16.100.2/32 172.16.100.2    100.100.100.5   dynamic  Tu1     <   >
       172.16.0.2/32 172.16.0.2      200.200.200.5   dynamic  Tu2     <   >
קטע קוד
HQ-ROUTER#show dmvpn 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
	N - NATed, L - Local, X - No Socket
	# Ent --> Number of NHRP entries with same NBMA peer
	NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
	UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel1, IPv4 NHRP Details 
Type:Hub, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 100.100.100.5      172.16.100.2    UP 04:29:11     D

Interface: Tunnel2, IPv4 NHRP Details 
Type:Hub, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 200.200.200.5        172.16.0.2    UP 04:28:18     D
🔄 3. High Availability & Path Tracking
VRRP State Validation
Verifying First Hop Redundancy Protocol (FHRP) behavior at the core/distribution layer for high-availability gateway redundancy:

קטע קוד
HQ-DIST-L3-SW1#show vrrp brief 
Interface        Grp Pri Time   Own Pre State    Master addr      Group addr
Vl10               1   150 3414       Y  Master  192.168.10.253  192.168.10.254 
Vl20               2   150 3414       Y  Master  192.168.20.253  192.168.20.254 
Vl30               3   150 3414       Y  Master  192.168.30.253  192.168.30.254 
Vl40               4   100 3609       Y  Backup  192.168.40.252  192.168.40.254 
Vl50               5   100 3609       Y  Backup  192.168.50.252  192.168.50.254 
IP SLA & Object Tracking Verification
Validating that the continuous tracking engine accurately evaluates the primary ISP link viability:

קטע קוד
HQ-ROUTER#show track 1
Track 1
  IP SLA 1 reachability
  Reachability is Up
    2 changes, last change 04:31:21
  Latest operation return code: OK
  Latest RTT (millisecs) 1
  Tracked by:
    Static IP Routing 0
קטע קוד
HQ-ROUTER#show ip sla statistics
IPSLA operation id: 1
	Latest RTT: 2 milliseconds
Latest operation start time: 23:53:24 EET Mon Jul 13 2026
Latest operation return code: OK
Number of successes: 197
Number of failures: 0
Operation time to live: Forever
