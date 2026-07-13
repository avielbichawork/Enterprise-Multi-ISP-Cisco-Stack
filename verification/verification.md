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
