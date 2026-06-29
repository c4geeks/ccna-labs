# Lab: ntp

Two Cisco IOS routers on one subnet, built to show NTP client and server
configuration, the stratum hierarchy, and NTP MD5 authentication, then to read
the verification output. Companion lab for:

- [How to Configure NTP on Cisco IOS (Client and Server)](https://computingforgeeks.com/cisco-ntp-configuration/)

## Topology

```
  [ R1 ]== Gi0/0  10.10.10.0/24  Gi0/0 ==[ R2 ]
   NTP master                          NTP client
   ntp master 3                        ntp server 10.10.10.1 key 1
   stratum 3                           stratum 4
```

- R1 is the authoritative time source: `ntp master 3` serves time from its own
  clock at stratum 3 (reference 127.127.1.1 / .LOCL.).
- R2 is the client: `ntp server 10.10.10.1 key 1` syncs to R1 and lands at
  stratum 4, one below the server.
- NTP MD5 authentication (key 1, `Cisc0NtpKey`) is configured on both ends, so
  R2 only trusts R1 once the shared key matches.

All addresses are RFC 1918 documentation ranges.

## Load the configs

Paste each file at the privileged-EXEC prompt of the matching device, in GNS3,
Cisco Packet Tracer, or on real gear:

- `configs/R1-config.txt` -> R1 (NTP master + authentication)
- `configs/R2-config.txt` -> R2 (NTP client, key 1)

There is no `.pkt` file: Packet Tracer's format cannot be generated outside the
app. Build the two routers, cable R1 Gi0/0 -> R2 Gi0/0, then paste.

## Verify

```
! On R2 (the client), after a few poll cycles:
show ntp status              ! Clock is synchronized, stratum 4, reference is 10.10.10.1
show ntp associations        ! * beside 10.10.10.1, reach climbing to 377
show ntp associations detail ! authenticated, our_master
show clock detail            ! Time source is NTP
```

NTP takes a few 64-second poll cycles to declare the clock synchronized; reach
climbs toward 377 as successive polls succeed.
