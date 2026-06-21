# Lab: ospf-single-area

Three routers in a line, all in OSPF area 0, built to demonstrate how single-area
OSPFv2 is configured and verified on Cisco IOS. Companion lab for:

- [Configure Single-Area OSPF on Cisco IOS](https://computingforgeeks.com/cisco-ospf-single-area-configuration/)

## Topology

```
   Internet
      |  (default route, originated by R1)
   [ R1 ] --- 10.0.12.0/30 --- [ R2 ] --- 10.0.23.0/30 --- [ R3 ]
  1.1.1.1                      2.2.2.2                      3.3.3.3
  LAN 10.1.1.0/24             LAN 10.2.2.0/24             LAN 10.3.3.0/24
  network method             interface method            network method
  + default-information
```

All three routers are in area 0. Each LAN is represented by a passive loopback so
it advertises as a clean /24; on real hardware that is the interface facing your
switch, configured identically.

## What this lab shows

- **Two ways to enable OSPF on an interface:** the `network` statement method
  (R1, R3) and the interface method `ip ospf 1 area 0` (R2).
- **Passive interfaces:** the LANs run no Hellos but are still advertised.
- **A default route into OSPF:** R1 has a default route and `default-information
  originate`, so R2 and R3 learn `O*E2 0.0.0.0/0`.
- **Verification:** FULL adjacencies, learned routes with `[110/cost]`, and cost.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (edge, network method, default origin)
- `configs/R2-config.txt` -> R2 (transit, interface method)
- `configs/R3-config.txt` -> R3 (branch, network method)

Wire R1 Gi0/0 to R2 Gi0/0, and R2 Gi1/0 to R3 Gi0/0.

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the app.

## Verify

```
R2# show ip ospf neighbor          ! 1.1.1.1 and 3.3.3.3 both FULL
R1# show ip protocols              ! router ID, networks, passive Lo1, ASBR
R2# show ip ospf interface brief   ! per-interface cost and DR/BDR role
R3# show ip route ospf             ! O routes + O*E2 0.0.0.0/0 default
R3# ping 10.1.1.1 source 10.3.3.3  ! 100% across the area
```

Give the routers ~60 seconds after boot for the adjacencies to reach FULL and the
SPF calculation to settle before the routes appear.
