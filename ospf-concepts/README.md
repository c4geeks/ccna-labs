# Lab: ospf-concepts

Three routers on one broadcast segment (via a switch), all in OSPF area 0, built
to show the OSPF concepts a CCNA candidate needs: neighbor adjacencies, DR/BDR
election, the link-state database, and cost. Companion lab for:

- [OSPF Concepts Explained for CCNA](https://computingforgeeks.com/ospf-concepts-explained/)

## Topology

```
                       [ SW1 ]   (L2 switch, 10.0.0.0/24 segment)
                     /    |    \
              Gi0/0 /  Gi0/1   \ Gi0/2
                  /      |       \
            [ R1 ]    [ R2 ]    [ R3 ]
          1.1.1.1    2.2.2.2    3.3.3.3
          DROTHER      BDR         DR
```

Router IDs 1.1.1.1 / 2.2.2.2 / 3.3.3.3 mean the highest (R3) is elected DR, R2 is
the BDR, and R1 is a DROTHER (all priorities default to 1, so the tie breaks on
router ID).

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (DROTHER)
- `configs/R2-config.txt` -> R2 (BDR)
- `configs/R3-config.txt` -> R3 (DR)

SW1 is a plain Layer 2 switch: every port in the default VLAN 1, no configuration
needed. Connect R1/R2/R3 Gi0/0 to three switch ports.

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the app.

## Verify

```
R1# show ip ospf neighbor             ! 3.3.3.3 FULL/DR, 2.2.2.2 FULL/BDR
R1# show ip ospf interface gi0/0      ! Network Type BROADCAST, cost, DR/BDR, Hello 10/Dead 40
R1# show ip ospf                      ! router ID, SPF runs, reference bandwidth, area 0
R1# show ip ospf database             ! 3 router (Type 1) LSAs + 1 network (Type 2) LSA from the DR
R1# show ip route ospf               ! O routes to the other loopbacks [110/2]
```

Give the segment ~40 seconds after boot: OSPF waits the dead interval (the "wait
timer") before settling the DR/BDR election.
