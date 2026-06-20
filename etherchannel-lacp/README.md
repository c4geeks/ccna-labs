# Lab: etherchannel-lacp

Two Layer 2 switches with two links bundled into one Port-channel using LACP.
Spanning Tree treats the bundle as a single link, so both members forward at once
instead of one being blocked. Companion lab for:

- [Configure EtherChannel (LACP) on Cisco Switches](https://computingforgeeks.com/cisco-etherchannel-configuration/)

## Topology

```
        Gi0/0 ============ Gi0/0
 [ SW1 ]      Port-channel 1      [ SW2 ]
        Gi0/1 ============ Gi0/1
           both links, LACP active
```

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching switch, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/SW1-config.txt` -> SW1
- `configs/SW2-config.txt` -> SW2

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two switches, wire Gi0/0-Gi0/0 and Gi0/1-Gi0/1, then paste.

## Verify

```
SW1# show etherchannel summary    ! Po1(SU), Protocol LACP, Gi0/0(P) Gi0/1(P)
SW1# show lacp neighbor           ! SW2 listed as the partner on both ports
SW1# show spanning-tree vlan 1    ! Po1 as ONE forwarding interface (no blocked port)
```

`(SU)` means a Layer 2 channel in use, and `(P)` means a member is bundled. Both
ends must use compatible LACP modes (active/active or active/passive); two passive
ends never form. Every member must match on speed, duplex, switchport mode, and
VLANs, so configure the `Port-channel 1` interface rather than the members. A
single flow rides one member link, so the bundle adds capacity across many flows,
not speed for one transfer.
