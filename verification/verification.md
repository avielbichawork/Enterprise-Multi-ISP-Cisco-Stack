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
 *>  200.200.200.4/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.0/30 200.200.200.2            0         200      0 200 i
 *>  209.165.100.4/30 200.200.200.2                      200      0 200 1000 i

Total number of prefixes 7
