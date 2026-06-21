# Lab: l3-switch-svi

Inter-VLAN routing on a Layer 3 switch using SVIs. The switch itself is the router:
one switched virtual interface per VLAN is the gateway, and the switch routes between
them in hardware, with no external router and no trunk to hairpin over. Companion lab for:

- [Configure Inter-VLAN Routing with a Layer 3 Switch (SVI)](https://computingforgeeks.com/cisco-layer3-switch-inter-vlan-svi/)

## Topology

```
                 [ SW1 ]  Layer 3 switch (ip routing)
                   |  |    SVI Vlan10 = 10.10.10.1 (VLAN 10 gw)
          Gi0/1 ___|  |___ Gi0/2   SVI Vlan20 = 10.20.20.1 (VLAN 20 gw)
              (vlan10)  (vlan20)
                |          |
           [ Host10 ]  [ Host20 ]
           10.10.10.10  10.20.20.20
```

No router. The Layer 3 switch terminates both VLANs on SVIs and routes between them.

## What this lab shows

- **ip routing** must be enabled (off by default on a multilayer switch).
- **SVIs** (`interface vlan 10`) are the per-VLAN gateways.
- **Autostate:** an SVI is up only when its VLAN has an active member port.
- **Verification:** connected SVI subnets in the routing table + a cross-VLAN ping.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/SW1-config.txt`     -> SW1 (Layer 3 switch: ip routing + SVIs + access ports)
- `configs/Host10-config.txt` -> Host10 (VLAN 10)
- `configs/Host20-config.txt` -> Host20 (VLAN 20)

Wire Host10 Gi0/0 to SW1 Gi0/1, Host20 Gi0/0 to SW1 Gi0/2.

The hosts are routers acting as end devices. On a real PC, just set the IP, mask, and
default gateway instead of pasting Host10/Host20.

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the app.

## Verify

```
SW1#    show ip route                 ! C 10.10.10.0/24 Vlan10, C 10.20.20.0/24 Vlan20
SW1#    show interfaces vlan 10       ! Vlan10 up/up, Hardware is Ethernet SVI, 10.10.10.1/24
SW1#    show vlan brief               ! Gi0/1 in vlan 10, Gi0/2 in vlan 20
Host10# ping 10.20.20.20              ! 100% across VLANs
Host10# traceroute 10.20.20.20       ! hop 1 = 10.10.10.1 (the SVI), hop 2 = Host20
```

The first ping after boot may be 60% while ARP resolves; the second is 100%.
The traceroute proves the switch itself is the router: the SVI is hop 1.
```
