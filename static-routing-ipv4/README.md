# Lab: static-routing-ipv4

Two routers with two links (a primary and a backup), built to show every IPv4
static route type on Cisco IOS and to demonstrate a floating static route failing
over to the backup link. Companion lab for:

- [Configure IPv4 Static Routes on Cisco (Default and Floating)](https://computingforgeeks.com/cisco-ipv4-static-routes-configuration/)

## Topology

```
                primary  10.0.12.0/30  (Gi0/0, AD 1)
        [ R1 ]=================================[ R2 ]
  LAN 192.168.1.0/24                       LAN 192.168.2.0/24
        [ R1 ]- - - - - - - - - - - - - - - - [ R2 ]
                backup  10.0.13.0/30  (Gi1/0, floating AD 200)
```

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (primary + floating static + default route)
- `configs/R2-config.txt` -> R2 (primary + floating static, so the return path also fails over)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two routers, connect Gi0/0 to Gi0/0 and Gi1/0 to Gi1/0, then paste.

## Verify (primary up)

```
R1# show ip route                          ! S 192.168.2.0/24 [1/0] via 10.0.12.2, S* default
R1# show ip route static                   ! just the static entries
R1# show ip route 192.168.2.0              ! distance 1 via the primary
R1# show running-config | include ip route ! all three statics incl the floating one
R1# ping 192.168.2.1 source 192.168.1.1    ! 100% over the primary
```

The floating static (AD 200) is configured but NOT in the routing table while the
primary (AD 1) is up. That is correct.

## Failover demo

Shut the primary interface on BOTH routers so the return path fails over too:

```
R1(config)# interface GigabitEthernet0/0
R1(config-if)# shutdown
R2(config)# interface GigabitEthernet0/0
R2(config-if)# shutdown
```

Then re-check on R1:

```
R1# show ip route 192.168.2.0              ! now distance 200 via 10.0.13.2 (floating installed)
R1# ping 192.168.2.1 source 192.168.1.1    ! still 100%, now over the backup link
```

The floating route installs because the primary's connected link is gone, so the
primary static (which recursed through it) is withdrawn and the higher-AD backup
becomes the best remaining route.
