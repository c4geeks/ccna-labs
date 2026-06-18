# Lab: dns-static

Two routers back to back. R1 holds static `ip host` name-to-IP records and R2
resolves names through R1 as a DNS client. `server1` maps to a loopback on R1
so the resolved address is genuinely reachable and pingable by name.

Article: [Configure DNS on Cisco Routers](https://computingforgeeks.com/ccna-labs-dns-server-configuration-on-gns3-and-packet-tracer/)

## Topology

```
        Gi0/0              Gi0/0
 [ R1 ]-------------------------[ R2 ]
 c7200                           c7200
 192.168.10.1/24       192.168.10.2/24
 Loopback0 10.0.0.1/32           (DNS client -> R1)
 (server1.lab.example.com)
```

## Addressing

| Device | Role | Interface | IP address |
|--------|------|-----------|------------|
| R1 | Name host | GigabitEthernet0/0 | 192.168.10.1/24 |
| R1 | server1 target | Loopback0 | 10.0.0.1/32 |
| R2 | DNS client | GigabitEthernet0/0 | 192.168.10.2/24 |

Name records on R1:

| Name | Address |
|------|---------|
| server1.lab.example.com | 10.0.0.1 |
| gw.lab.example.com | 192.168.10.1 |

## Files

- [`configs/R1-dns.txt`](configs/R1-dns.txt)
- [`configs/R2-dns-client.txt`](configs/R2-dns-client.txt)
- [`topology.json`](topology.json)

## A note on `ip dns server`

Making a router answer DNS queries for clients (a real resolver) needs the
`ip dns server` command, which exists only on ISR, IOSv, and IOS-XE. On a c7200
(dynamips) it returns `% Invalid input detected at '^' marker.`. This lab
therefore demonstrates what is universally testable: static `ip host`
resolution plus DNS-client config. The article covers the `ip dns server` path
honestly, including the platforms it requires.

## Load it

Connect R1 Gi0/0 to R2 Gi0/0, open each console, and paste the matching config.
In Packet Tracer use two routers (e.g. 4321) and paste the same commands.

## Verify

On R2, resolution and reachability by name:

```
ping server1.lab.example.com
show hosts
```

The ping should resolve `server1.lab.example.com` to `10.0.0.1` and succeed.
`show hosts` on R1 lists the permanent static entries.
