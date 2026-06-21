# Lab: router-on-a-stick

Inter-VLAN routing with router-on-a-stick: one router routes between two VLANs over
a single 802.1Q trunk to a switch, using one subinterface per VLAN. Companion lab for:

- [Router-on-a-Stick Inter-VLAN Routing on Cisco](https://computingforgeeks.com/cisco-router-on-a-stick-inter-vlan-routing/)

## Topology

```
            [ R1 ]  Gi0/0.10 = 10.10.10.1 (VLAN 10 gw)
              |     Gi0/0.20 = 10.20.20.1 (VLAN 20 gw)
              |  Gi0/0  (802.1Q trunk, VLANs 10,20)
            [ SW1 ]
           /       \
   Gi0/1 (vlan10)   Gi0/2 (vlan20)
        |               |
   [ Host10 ]       [ Host20 ]
   10.10.10.10      10.20.20.20
```

The router's physical Gi0/0 carries no IP; the subinterfaces own the VLAN gateways.
The switch port to the router is an 802.1Q trunk; the host ports are access ports.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt`     -> R1 (router-on-a-stick, subinterfaces)
- `configs/SW1-config.txt`    -> SW1 (trunk + access ports)
- `configs/Host10-config.txt` -> Host10 (VLAN 10)
- `configs/Host20-config.txt` -> Host20 (VLAN 20)

Wire R1 Gi0/0 to SW1 Gi0/0 (trunk), Host10 Gi0/0 to SW1 Gi0/1, Host20 Gi0/0 to SW1 Gi0/2.

The hosts are routers acting as end devices. On a real PC, just set the IP, mask, and
default gateway instead of pasting Host10/Host20.

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the app.

## Verify

```
R1#     show ip interface brief     ! Gi0/0 no IP, Gi0/0.10 and .20 up with gateways
R1#     show ip route               ! C 10.10.10.0/24 and C 10.20.20.0/24 (both connected)
R1#     show vlans                  ! each subinterface mapped to its 802.1Q VLAN
SW1#    show interfaces trunk       ! Gi0/0 trunking 802.1q, vlans 10,20
Host10# ping 10.20.20.20            ! 100% across VLANs
Host10# traceroute 10.20.20.20      ! hop 1 = 10.10.10.1 (R1), hop 2 = Host20
```

The traceroute is the proof: inter-VLAN traffic goes up to the router and back down.
```
