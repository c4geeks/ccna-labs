# Lab: stp-rapid-pvst

Three Layer 2 switches wired into a triangle, which is one physical loop. Rapid
PVST+ elects a root bridge and blocks exactly one port to break the loop, leaving
the redundant link ready to take over. Companion lab for:

- [Spanning Tree Protocol (Rapid PVST+) Configuration on Cisco Switches](https://computingforgeeks.com/cisco-spanning-tree-protocol-configuration/)

## Topology

```
                 [ SW1 ]  root, priority 4096
                Gi0/0  Gi0/1
               /            \
        forwarding        forwarding
             /                \
        Gi0/0                Gi0/0
      [ SW2 ]--- Gi0/1 -- blocked -- Gi0/1 ---[ SW3 ]
   priority 8192     (SW3 blocks)      priority 32768
```

SW1 wins the root election (lowest priority). SW3 keeps the default priority, so
on the SW2-SW3 segment SW2 is designated and STP puts SW3's Gi0/1 into the
blocking (alternate) state. SW3's root port is Gi0/0 toward SW1.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching switch, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/SW1-config.txt` -> SW1 (root)
- `configs/SW2-config.txt` -> SW2 (secondary)
- `configs/SW3-config.txt` -> SW3 (default)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the three switches, wire Gi0/0-Gi0/1 into the triangle, then paste.

## Verify

```
SW1# show spanning-tree vlan 1     ! This bridge is the root, all ports Desg FWD
SW3# show spanning-tree vlan 1     ! Gi0/0 Root FWD, Gi0/1 Altn BLK (loop broken)
SW3# show spanning-tree root       ! Root ID 4097, cost 4, root port Gi0/0
SW1# show spanning-tree summary    ! Switch is in rapid-pvst mode, pathcost short
```

The blocked port (`Altn BLK` on SW3 Gi0/1) is the loop being broken. Pull the
SW1-SW3 cable and SW3 unblocks Gi0/1 within seconds, which is the rapid in Rapid
PVST+. PortFast and BPDU Guard on SW1 Gi0/3 protect a host-facing access port:
PortFast forwards immediately, BPDU Guard err-disables it if a switch is ever
plugged in there.
