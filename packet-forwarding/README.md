# Lab: packet-forwarding

Two routers over an OSPF area 0 link, plus a static route on R1, built so you can
watch how a Cisco router actually forwards a packet: the CEF forwarding table
(FIB), the adjacency table (the Layer 2 rewrite), ARP, and recursion. Companion
lab for:

- [How a Cisco Router Forwards a Packet (CEF, FIB, Adjacency)](https://computingforgeeks.com/cisco-router-packet-forwarding-explained/)

## Topology

```
 [ R1 ]---- Gi0/0 -- 10.0.12.0/30 -- Gi0/0 ----[ R2 ]
  Lo0 1.1.1.1/32        OSPF area 0       Lo0 2.2.2.2/32
  Lo1 172.16.1.0/24                       Lo1 192.168.20.0/24
  + static 198.51.100.0/24 via 10.0.12.2 (recursion)
```

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (OSPF + a static route via a next-hop IP)
- `configs/R2-config.txt` -> R2 (advertises its loopbacks into OSPF)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two routers, connect Gi0/0 to Gi0/0, then paste.

## Verify the forwarding machinery

```
R1# ping 192.168.20.1 source 172.16.1.1      ! prove the path forwards
R1# show ip cef                              ! the FIB: receive/attached/next-hop/drop
R1# show ip cef 192.168.20.1                 ! one destination, resolved
R1# show ip cef 198.51.100.1                 ! the static route, recursion resolved
R1# show adjacency GigabitEthernet0/0 detail ! the Layer 2 rewrite (Encap length 14)
R1# show ip arp                              ! where the destination MAC came from
R1# show ip route 192.168.20.0              ! the RIB entry the FIB is built from
```

The adjacency rewrite string is the 14-byte Ethernet header the router pushes:
6 bytes destination MAC (the next hop), 6 bytes source MAC (R1's own interface),
and 2 bytes EtherType (0800 for IPv4). Decode it and match each MAC against
`show ip arp` to see how the FIB, the adjacency table, and ARP fit together.
The loopbacks use `ip ospf network point-to-point` so OSPF advertises their real
mask (a /24) instead of a /32 host route.
