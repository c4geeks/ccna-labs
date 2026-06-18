# Lab: base-device

A router and a Layer 2 switch on one management subnet, hardened with a hashed
enable secret, a local admin user, an MOTD banner, and SSH-only remote access.
This is the foundation lab for two articles:

- [Cisco Device Base Configuration](https://computingforgeeks.com/cisco-device-base-configuration/)
- [Configure SSH on Cisco Routers and Switches](https://computingforgeeks.com/ccna-labs-ssh-access-configuration-on-gns3-and-packet-tracer/)

## Topology

```
        Gi0/0                Gi0/0
 [ R1 ]-----------------------------[ SW1 ]
 c7200                                IOSvL2
 192.168.10.1/24            192.168.10.2/24 (Vlan1)
```

## Addressing

| Device | Role | Interface | IP address |
|--------|------|-----------|------------|
| R1 | Router | GigabitEthernet0/0 | 192.168.10.1/24 |
| SW1 | L2 switch | Vlan1 (SVI) | 192.168.10.2/24 |

Credentials (lab only, change before real use):

- enable secret: `Cisc0-Lab!`
- local user: `admin` / `Adm1n-Lab!` (privilege 15)
- console password: `Con-Lab!`

## Files

- [`configs/R1-base-config.txt`](configs/R1-base-config.txt)
- [`configs/SW1-base-config.txt`](configs/SW1-base-config.txt)
- [`topology.json`](topology.json)

## Load it

**GNS3 / EVE-NG / real gear:** add a router and an L2 switch, connect R1 Gi0/0
to SW1 Gi0/0, open each console, and paste the matching `*.txt`. Both files
include `configure terminal` and `write memory`.

**Packet Tracer:** drop a router (e.g. 4321/2911) and a 2960 switch, link them,
then paste the same configs into each device CLI. RSA key generation works in
Packet Tracer; if a specific platform rejects `crypto key generate rsa modulus
2048`, run `crypto key generate rsa` and enter `2048` at the prompt.

## Verify

On R1 and SW1:

```
show ip ssh
show ip interface brief
show users
```

Then prove SSH works and Telnet is refused from one device to the other:

```
ssh -l admin 192.168.10.2
telnet 192.168.10.2
```

SSH should prompt for the `admin` password; Telnet should return
`% Connection refused by remote host` because `transport input ssh` allows SSH
only.
