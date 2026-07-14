# 🔍 Infrastructure Verification Guide

This document outlines the systematic verification commands used to validate control plane stability, overlay infrastructure health, and deterministic data plane forwarding behavior across the multi-vendor environment.

---

## 🌐 1. Dynamic Routing Verification

### Underlay BGP Peer Validation & Enterprise Core EIGRP
Below are the live verification outputs capturing active BGP peerings with service providers (ISP-1/ISP-2) and interior EIGRP Named-Mode routing convergence across the core:

```text
==========================================================================
HQ-ROUTER# show ip bgp summary
==========================================================================
BGP router identifier 200.200.200.1, local AS number 500
BGP table version is 10, main routing table version 10
7 network entries using 980 bytes of memory
12 path entries using 960 bytes of memory
7/4 BGP path/bestpath attribute entries using 1008 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
BGP using 3044 total bytes of memory

Neighbor        V        AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down    State/PfxRcd
100.100.100.2   4       100     296     296        10    0    0 04:23:32         5
200.200.200.2   4       200     296     297        10    0    0 04:23:42         5


==========================================================================
HQ-ROUTER# show ip bgp neighbors 200.200.200.2 advertised-routes
==========================================================================
     Network          Next Hop            Metric LocPrf Weight Path
 *>  8.8.8.8/32       200.200.200.2                      200      0 200 1000 i
 *>  100.100.100.0/30 0.0.0.0                  0         32768 i
 *>  100.100.100.4/30 100.100.100.2            0             0 100 i
 *>  200.200.200.0/30 0.0.0.0                  0         32768 i
 *>  200.200.200.4/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.0/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.4/30 200.200.200.2                      200      0 200 1000 i

Total number of prefixes 7 


==========================================================================
HQ-ROUTER# show ip eigrp neighbors
==========================================================================
EIGRP-IPv4 VR(NAME-1) Address-Family Neighbors for AS(1)
H   Address                 Interface           Hold Uptime   SRTT   RTO  Q  Seq
                                                (sec)         (ms)       Cnt Num
1   10.0.0.5                Et0/0                    12 04:25:21    8   100  0  34
0   10.0.0.1                Et0/1                    12 04:25:21    6   100  0  33


==========================================================================
HQ-ROUTER# show ip eigrp topology
==========================================================================
EIGRP-IPv4 VR(NAME-1) Topology Table for AS(1)/ID(200.200.200.1)

P 192.168.10.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 172.16.0.2/32, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.110.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 172.16.100.0/24, 1 successors, FD is 65542553600, tag is 100
        via Redistributed (65542553600/0)
P 192.168.30.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
P 0.0.0.0/0, 1 successors, FD is 131072000
        via Rstatic (131072000/0)
P 192.168.50.0/24, 2 successors, FD is 131727360
        via 10.0.0.1 (131727360/1310720), Ethernet0/1
        via 10.0.0.5 (131727360/1310720), Ethernet0/0
