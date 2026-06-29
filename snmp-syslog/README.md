# Lab: snmp-syslog

One monitored Cisco router sending Syslog messages and SNMP traps to a
monitoring host, built to show Syslog severity levels, a remote logging host,
and SNMP in both v2c and v3. Companion lab for:

- [How to Configure SNMP and Syslog on Cisco IOS](https://computingforgeeks.com/cisco-snmp-syslog-configuration/)

## Topology

```
  [ R1 ]== Gi0/0  10.20.20.0/24  Gi0/0 ==[ NMS ]
   monitored router                      Syslog + SNMP collector
   10.20.20.1                            10.20.20.50
   logging host + snmp-server            (rsyslog/Graylog + LibreNMS/Zabbix)
```

- R1 sends Syslog to 10.20.20.50 (UDP 514) at trap level `informational` and
  SNMP traps (UDP 162) to the same host.
- SNMP is configured two ways on R1: v2c with a read-only community `CFG-RO`,
  and v3 with an authPriv user `cfgmon` (SHA authentication, AES encryption).
- NMS is a stand-in for the collector so R1 has a reachable destination; in
  production it is a real Syslog/SNMP platform, not a router.

All addresses are RFC 1918 documentation ranges.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt`  -> R1 (monitored device: Syslog + SNMP v2c + v3)
- `configs/NMS-config.txt` -> NMS (collector stand-in, just an IP)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two devices, cable R1 Gi0/0 -> NMS Gi0/0, then paste.

## Verify (on R1)

```
show logging              ! trap level + logging host + the buffered messages
show snmp community       ! the v2c community CFG-RO
show snmp user            ! the v3 user cfgmon (Auth SHA, Priv AES128)
show snmp host            ! both trap destinations, v2c and v3 priv
show snmp                 ! chassis, contact, location, and packet counters
```

Bounce an interface (`shutdown` / `no shutdown` on a loopback) to generate real
log messages, then re-run `show logging` to see them in the buffer.
