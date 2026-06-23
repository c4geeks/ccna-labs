# Lab: dhcp-server-relay

A central Cisco IOS DHCP server and a DHCP relay agent, built to show both halves
of Domain 4 DHCP: configuring a router as the DHCP server, and relaying requests
across a router so one server covers a subnet it is not attached to. Companion lab
for:

- [Configure a Cisco DHCP Server and Relay](https://computingforgeeks.com/cisco-dhcp-server-relay-configuration/)

## Topology

```
  ClientA 192.168.10.11 (direct)                 ClientB 192.168.20.11 (via relay)
        |  192.168.10.0/24                                |  192.168.20.0/24
      [ R1 ]==== 10.0.0.0/30 ====[ R2 ]
   DHCP server                  DHCP relay
   LAN10 + LAN20 pools          ip helper-address 10.0.0.1
```

- R1 is the DHCP server with a pool per LAN and a route to 192.168.20.0/24 via R2.
- R2 has no pool; its client interface relays DHCP to R1 with `ip helper-address`.
- ClientA leases directly; ClientB's request is relayed to R1.

All addresses are RFC 1918 private ranges. `8.8.8.8` is used only as an example
public DNS resolver.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt`      -> R1 (DHCP server, two pools)
- `configs/R2-config.txt`      -> R2 (DHCP relay, ip helper-address)
- `configs/ClientA-config.txt` -> ClientA (DHCP client on R1's LAN)
- `configs/ClientB-config.txt` -> ClientB (DHCP client on R2's LAN)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the four devices, cable ClientA->R1 Gi0/0, R1 Gi1/0->R2 Gi0/0, and
R2 Gi1/0->ClientB Gi0/0, then paste.

## Verify

```
! On R1 (server):
show ip dhcp binding
show ip dhcp pool

! On each client:
show ip interface brief
show dhcp lease
```

ClientA's lease names server `192.168.10.1` (the local gateway). ClientB's lease
names server `10.0.0.1`, the server one hop away, which is the proof its request
crossed the relay. On R1, the relayed binding (192.168.20.11) shows its interface
as `Unknown`, while the direct binding (192.168.10.11) shows `GigabitEthernet0/0`.

## Relay note

A router does not forward broadcasts between subnets, so a client cannot reach a
DHCP server on another subnet on its own. `ip helper-address <server>` on the
client-facing interface turns those broadcasts into a unicast to the server, and
the relay stamps in the gateway address (giaddr) so the server picks the right
pool. The same command also forwards a default list of other broadcast UDP
services (TFTP, DNS, TACACS, NTP, NetBIOS, BOOTP); tune it with
`ip forward-protocol`.
