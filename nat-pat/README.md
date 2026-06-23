# Lab: nat-pat

One NAT router and an outside router, built to show all three inside-source NAT
forms on Cisco IOS, static NAT, dynamic NAT with a pool, and PAT (overload), and
to read the resulting translation table. Companion lab for:

- [Configure NAT and PAT on a Cisco Router](https://computingforgeeks.com/cisco-nat-pat-configuration/)

## Topology

```
  Host1 192.168.10.10                              198.51.100.1 (Lo0)
        |  192.168.10.0/24 (inside)                       |
      [ R1 ]== Gi1/0 203.0.113.1/30 (outside) ==[ R2 ]  internet stand-in
        |  192.168.20.0/24 (inside)
  Host2 192.168.20.20  (inside server, static NAT -> 203.0.113.10)
```

- R1 Gi0/0 and Gi2/0 are `ip nat inside`; Gi1/0 is `ip nat outside`.
- Static NAT: 192.168.20.20 (Host2) maps permanently to 203.0.113.10.
- PAT overload: the 192.168.10.0/24 LAN (Host1) overloads onto pool PUBLIC
  (203.0.113.9), each flow getting a translated port.
- R2 routes the public NAT block 203.0.113.8/29 back to R1.

All addresses are documentation ranges (RFC 1918 private, RFC 5737 public).

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt`    -> R1 (NAT router: inside/outside, static + pool + overload)
- `configs/R2-config.txt`    -> R2 (internet stand-in + route back)
- `configs/Host1-config.txt` -> Host1 (PAT LAN host)
- `configs/Host2-config.txt` -> Host2 (inside server, static NAT)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the four devices, cable Host1->R1 Gi0/0, Host2->R1 Gi2/0, and
R1 Gi1/0->R2 Gi0/0, then paste.

## Verify

```
! On Host1 (PAT) and Host2 (static), reach the outside host:
ping 198.51.100.1

! On R1, read the translations and statistics:
show ip nat translations
show ip nat statistics
```

You should see a PAT entry whose inside global carries a translated port
(`icmp 203.0.113.9:1024 192.168.10.10:0 ...`) and a permanent static entry with
no port (`--- 203.0.113.10 192.168.20.20 --- ---`). `show ip nat statistics`
reports the inside/outside interfaces, the pool at 100% allocation, and a rising
hit count with zero misses.

## Pool gotcha

The pool deliberately starts at `203.0.113.9`, not `.8`. Because the pool is
defined with `prefix-length 29`, IOS reserves the `.8` network and `.15`
broadcast of that subnet by default and will not allocate them. A pool starting
at `.8` never allocates: `show ip nat statistics` shows misses climbing while
`allocated 0`, and the PAT host's pings fail.
