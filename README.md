# CCNA 200-301 Labs

Copy-paste device configurations and topologies for the hands-on labs in the
[ComputingForGeeks CCNA 200-301 series](https://computingforgeeks.com/quickly-prepare-for-ccna-200-301-exam/).

Every configuration in this repo was built and tested on real Cisco IOS images
(IOSv / IOSvL2 / c7200) in GNS3, then captured into the matching article. The
files here are the exact, working configs, ready to paste into **GNS3**,
**Cisco Packet Tracer**, or **real Cisco gear**.

## Labs

| Lab | Topology | Used by article |
|-----|----------|-----------------|
| [`base-device/`](base-device/) | R1 (router) + SW1 (L2 switch) | [Cisco Device Base Configuration](https://computingforgeeks.com/cisco-device-base-configuration/) and [Configure SSH on Cisco Routers and Switches](https://computingforgeeks.com/ccna-labs-ssh-access-configuration-on-gns3-and-packet-tracer/) |
| [`dns-static/`](dns-static/) | R1 (DNS host) + R2 (DNS client) | [Configure DNS on Cisco Routers](https://computingforgeeks.com/ccna-labs-dns-server-configuration-on-gns3-and-packet-tracer/) |
| [`hsrp/`](hsrp/) | R1 + R2 (HSRP pair) + SW1 | [Configure HSRP on Cisco Routers](https://computingforgeeks.com/ccna-labs-hsrp-configuration-on-gns3-and-packet-tracer/) |

## How to use these

Each lab folder has a `README.md` with the topology, an addressing table, and
the load steps, plus one paste-ready `*.txt` config per device and a
`topology.json` describing the nodes and links.

- **Real Cisco gear / GNS3 / EVE-NG:** open the device console, then paste the
  matching `*.txt` at the prompt. Configuration mode and `write memory` are
  included in each file.
- **Cisco Packet Tracer:** Packet Tracer uses its own encrypted `.pkt` format
  that cannot be generated outside the program, so there is no `.pkt` here.
  Instead, drop the devices onto the canvas, open each device CLI, and paste the
  same `*.txt`. The commands are standard IOS and work unchanged in Packet
  Tracer (subject to the small command subset Packet Tracer supports, noted per
  lab where it matters).

## Notes

- Passwords in these files (`Cisc0-Lab!`, `Adm1n-Lab!`, `Con-Lab!`) are
  throwaway lab values. Change them before using any of this outside a lab.
- Addresses are RFC 1918 (`192.168.10.0/24`, `10.0.0.1`) so nothing here
  collides with the public internet.
- `crypto key generate rsa` takes a few seconds; on real gear, let it finish
  before pasting the rest of a block.

Companion repo for the ComputingForGeeks CCNA 200-301 learning series.
Licensed MIT (see [`LICENSE`](LICENSE)).
