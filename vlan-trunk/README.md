# Lab: vlan-trunk

Two Layer 2 switches sharing VLAN 10 (Sales) and VLAN 20 (Engineering) over an
802.1Q trunk. SW1 has an access port in VLAN 10, SW2 has an access port in
VLAN 20, and the trunk between them carries both VLANs so each VLAN spans the
two switches. This is the foundation lab for:

- [Configure VLANs on a Cisco Switch](https://computingforgeeks.com/cisco-vlans-configuration-guide/)
- 802.1Q trunking between switches (companion guide)

## Topology

```
                 Gi0/0          Gi0/0
 [ Sales PC ]---[ SW1 ]==================[ SW2 ]---[ Engineering PC ]
   VLAN 10    Gi0/1    802.1Q trunk        Gi0/2        VLAN 20
              access   (VLAN 10 + 20)      access
```

## VLANs

| VLAN | Name        | SW1 access port | SW2 access port |
|------|-------------|-----------------|-----------------|
| 10   | SALES       | Gi0/1           | (trunk only)    |
| 20   | ENGINEERING | (trunk only)    | Gi0/2           |

Both VLANs exist on both switches; the Gi0/0 trunk carries them between SW1 and SW2.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching switch, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/SW1-vlan-config.txt` -> SW1
- `configs/SW2-vlan-config.txt` -> SW2

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two switches, connect Gi0/0 to Gi0/0, then paste the configs.

## Verify

```
SW1# show vlan brief          ! VLAN 10 active on Gi0/1, VLAN 20 active
SW1# show interfaces gi0/1 switchport   ! static access, Access Mode VLAN 10
SW1# show interfaces trunk    ! Gi0/0 trunking, VLANs 1,10,20 active
SW2# show vlan brief          ! VLAN 20 active on Gi0/2, VLAN 10 active
SW2# show interfaces trunk    ! Gi0/0 trunking, VLANs 1,10,20 active
```

Access ports read `Operational Mode: down` until a host is cabled to them; the
configuration is correct regardless. Devices in VLAN 10 cannot reach VLAN 20
without a router or a Layer 3 switch (inter-VLAN routing, covered separately).
