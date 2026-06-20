# Lab: routing-table

Two routers over an OSPF area 0 link, built so that `show ip route` on R1 shows
every route code the CCNA cares about in one table: connected (C), local (L),
OSPF (O), static (S), and the default route (S*). Companion lab for:

- [How to Read the Cisco IP Routing Table (show ip route)](https://computingforgeeks.com/cisco-ip-routing-table-explained/)

## Topology

```
 [ R1 ]---- Gi0/0 -- 10.0.12.0/30 -- Gi0/0 ----[ R2 ]
  Lo0 1.1.1.1/32        OSPF area 0       Lo0 2.2.2.2/32
  Lo1 172.16.1.0/24                       Lo1 192.168.20.0/24
  + static 203.0.113.0/24 via 10.0.12.2
  + default 0.0.0.0/0 via 10.0.12.2
```

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (OSPF + a static + a default route)
- `configs/R2-config.txt` -> R2 (advertises its loopbacks into OSPF)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two routers, connect Gi0/0 to Gi0/0, then paste.

## Verify

```
R1# show ip route               ! C, L, O [110/x], S [1/0], and S* default
R1# show ip route 192.168.20.0  ! the full record: source, distance, metric
R1# show ip ospf neighbor       ! 2.2.2.2 in FULL state
```

The bracket on each learned route is [administrative distance / metric]. The
loopbacks use `ip ospf network point-to-point` so OSPF advertises their real
mask (a /24) instead of the default /32 host route.
