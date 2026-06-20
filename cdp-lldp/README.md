# Lab: cdp-lldp

Two Layer 2 switches discovering each other with the two CCNA discovery
protocols: CDP (Cisco, on by default) and LLDP (the IEEE 802.1AB open standard,
which you turn on with `lldp run`). Companion lab for:

- [CDP and LLDP Network Discovery on Cisco Devices](https://computingforgeeks.com/cisco-cdp-lldp-network-discovery/)

## Topology

```
 [ SW1 ]---------------------[ SW2 ]
  Gi0/0    CDP + LLDP link    Gi0/0
 10.10.10.1                 10.10.10.2
```

Each switch has a `Vlan1` SVI for a management IP so that `show cdp neighbors
detail` and `show lldp neighbors detail` display the neighbor's address.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching switch, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/SW1-config.txt` -> SW1
- `configs/SW2-config.txt` -> SW2

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two switches, connect Gi0/0 to Gi0/0, then paste the configs.

## Verify

```
SW1# show cdp neighbors            ! SW2 on Gi0/0, capability S I, platform vios_l2
SW1# show cdp neighbors detail     ! adds SW2 IP 10.10.10.2, IOS version, native VLAN
SW1# show lldp neighbors           ! SW2 on Gi0/0 (only after lldp run on both)
SW1# show lldp neighbors detail    ! adds SW2 system name, description, mgmt IP
SW2# show cdp                      ! CDP sends every 60s, holdtime 180s
SW2# show lldp                     ! LLDP sends every 30s, holdtime 120s
```

CDP is on by default, so `show cdp neighbors` works with no configuration. LLDP
stays empty until `lldp run` is set on both ends. On edge or untrusted ports,
turn discovery off with `no cdp enable` and `no lldp transmit` / `no lldp
receive` so you do not advertise device details to whatever is plugged in.
