# 🔍 Infrastructure Verification Guide

This document outlines the systematic verification commands used to validate control plane stability, overlay infrastructure health, and deterministic data plane forwarding behavior across the multi-vendor environment.

---

## 🌐 1. Dynamic Routing Verification

### Underlay BGP Peer Validation
To verify that the Edge Routers successfully establish eBGP peerings with the service providers (ISP-1 and ISP-2) and receive correct routing advertisements:

```baseline
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

Neighbor        V        AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down    State/PfxRcd
100.100.100.2   4       100     296     296        10    0    0 04:23:32         5
200.200.200.2   4       200     296     297        10    0    0 04:23:42         5
