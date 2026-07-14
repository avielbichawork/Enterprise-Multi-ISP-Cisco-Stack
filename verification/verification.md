# 🔍 Infrastructure Verification Guide

This document outlines the systematic verification commands used to validate control plane stability, overlay infrastructure health, and deterministic data plane forwarding behavior across the multi-vendor environment.

---

## 🌐 1. Dynamic Routing Verification

### Underlay BGP Peer Validation
To verify that the Edge Routers successfully establish eBGP peerings with the service providers (ISP-1 and ISP-2) and receive correct routing advertisements:

#### 💻 HQ-ROUTER# `show ip bgp summary`
```text
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
💻 HQ-ROUTER# show ip bgp neighbors 200.200.200.2 advertised-routes
Plaintext
BGP table version is 10, local router ID is 200.200.200.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal

     Network          Next Hop            Metric LocPrf Weight Path
 *>  8.8.8.8/32       200.200.200.2                      200      0 200 1000 i
 *>  100.100.100.0/30 0.0.0.0                  0         32768 i
 *>  100.100.100.4/30 100.100.100.2            0             0 100 i
 *>  200.200.200.0/30 0.0.0.0                  0         32768 i
 *>  200.200.200.4/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.0/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.4/30 200.200.200.2                      200      0 200 1000 i

Total number of prefixes 7 
Enterprise Core EIGRP Validation (Named-Mode)
Validating EIGRP interior routing convergence across the campus core and distribution layers:

💻 HQ-ROUTER# show ip eigrp neighbors
Plaintext
EIGRP-IPv4 VR(NAME-1) Address-Family Neighbors for AS(1)
H   Address                 Interface           Hold Uptime   SRTT   RTO  Q  Seq
                                                (sec)         (ms)       Cnt Num
1   10.0.0.5                Et0/0                    12 04:25:21    8   100  0  34
0   10.0.0.1                Et0/1                    12 04:25:21    6   100  0  33
