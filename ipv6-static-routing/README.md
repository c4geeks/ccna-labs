# Lab: ipv6-static-routing

Two routers with two links (a primary and a backup), built to show IPv6 static
routing on Cisco IOS, including the parts that differ from IPv4: enabling IPv6
routing, the link-local next-hop rule, the `::/0` default, and a floating static
that fails over. Companion lab for:

- [Configure IPv6 Static Routes on Cisco (link-local, default, floating)](https://computingforgeeks.com/cisco-ipv6-static-routes-configuration/)

## Topology

```
                primary  2001:DB8:12::/64  (Gi0/0, AD 1)
        [ R1 ]=================================[ R2 ]
   2001:DB8:1::/64                          2001:DB8:2::/64
        [ R1 ]- - - - - - - - - - - - - - - - [ R2 ]
                backup  2001:DB8:13::/64  (Gi1/0, floating AD 200)
```

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching router, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (ipv6 unicast-routing + link-local primary + floating + ::/0 default)
- `configs/R2-config.txt` -> R2 (so the return path also fails over)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two routers, connect Gi0/0 to Gi0/0 and Gi1/0 to Gi1/0, then paste.

## The link-local next-hop rule

A link-local (`FE80::`) next hop is only unique on one link, so IOS rejects it
without an exit interface:

```
R1(config)# ipv6 route 2001:DB8:2::/64 FE80::2
% Interface has to be specified for a link-local nexthop
```

Use the fully-specified form instead: `ipv6 route 2001:DB8:2::/64 GigabitEthernet0/0 FE80::2`.

## Verify (primary up)

```
R1# show ipv6 route                          ! S ::/0, S 2001:DB8:2::/64 [1/0] via FE80::2 Gi0/0
R1# show ipv6 route 2001:DB8:2::/64          ! distance 1 + Backup from "static [200]"
R1# show ipv6 interface brief               ! the FE80:: link-locals to reference
R1# ping 2001:DB8:2::1 source 2001:DB8:1::1 ! 100% over the primary
```

## Failover demo

Shut the primary interface on BOTH routers so the return path fails over too:

```
R1(config)# interface GigabitEthernet0/0
R1(config-if)# shutdown
R2(config)# interface GigabitEthernet0/0
R2(config-if)# shutdown
```

Then on R1, `show ipv6 route 2001:DB8:2::/64` shows the floating route installed at
`distance 200` via `FE80::20, GigabitEthernet1/0`, and the ping still succeeds over
the backup. `ipv6 unicast-routing` must be enabled on both routers or nothing forwards.
