# Lab: hsrp

Two routers present one virtual default gateway to the LAN using HSRP. R1 wins
the active role (priority 110 + preempt); R2 stands by (priority 100) and takes
over only when R1 fails, then hands the role back when R1 returns.

Article: [Configure HSRP on Cisco Routers](https://computingforgeeks.com/ccna-labs-hsrp-configuration-on-gns3-and-packet-tracer/)

## Topology

```
 [ R1 ]  Gi0/0 ---- Gi0/1 \
 192.168.10.1/24          \
 pri 110, preempt          [ SW1 ]  (Layer 2, no config)
                          /
 [ R2 ]  Gi0/0 ---- Gi0/2 /
 192.168.10.2/24
 pri 100, preempt

 Virtual gateway (HSRP group 1, v2): 192.168.10.254
```

## Addressing

| Device | Interface | IP address | HSRP priority | Role |
|--------|-----------|------------|---------------|------|
| R1 | GigabitEthernet0/0 | 192.168.10.1/24 | 110 (preempt) | Active |
| R2 | GigabitEthernet0/0 | 192.168.10.2/24 | 100 (preempt) | Standby |
| Virtual IP | (group 1) | 192.168.10.254 | n/a | Gateway hosts use this |
| SW1 | switchports Gi0/1, Gi0/2 | none | n/a | Layer 2 only |

SW1 needs no configuration: a default Layer 2 switch already forwards the
group's frames in VLAN 1. Connect R1 Gi0/0 to SW1 Gi0/1 and R2 Gi0/0 to SW1
Gi0/2.

## Files

- [`configs/R1-hsrp.txt`](configs/R1-hsrp.txt)
- [`configs/R2-hsrp.txt`](configs/R2-hsrp.txt)
- [`topology.json`](topology.json)

## Load it

Build the two routers and a switch, wire them as above, and paste each router
config. Packet Tracer supports HSRP on its router platforms, so the same
commands work there.

## Verify

```
show standby brief
show standby
```

On R1 you should see it as `Active` with the virtual MAC `0000.0c9f.f001`
(HSRP v2, group 1) and the virtual IP `192.168.10.254`. R2 shows `Standby`.

## Failover drill

```
! on R1
interface GigabitEthernet0/0
 shutdown
```

Within the hold time R2 becomes `Active`. Bring R1 back:

```
! on R1
interface GigabitEthernet0/0
 no shutdown
```

Because R1 has `preempt`, it reclaims the active role automatically. Pinging
`192.168.10.254` from a LAN host throughout the drill shows at most one or two
dropped replies during the switchover.
