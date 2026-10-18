# nokia-huawei-cli

**Side-by-side Nokia SR OS ↔ Huawei VRP command reference for IP/MPLS operations and troubleshooting.**

> 🇹🇷 Aynı işi yapan Nokia SR OS ve Huawei VRP komutlarını yan yana gösteren, sahada kullanılmak için hazırlanmış bir başvuru kaynağı. İki üreticinin bir arada bulunduğu operatör ağlarında çalışanlar için.

If you operate a multi-vendor carrier network, you constantly translate between two CLIs in your head. This repository does that translation for the commands you use every day.

- **Nokia** examples use the **classic CLI** (SR OS on 7750 SR / 7450 ESS / 7950 XRS).
- **Huawei** examples use **VRP** on NE40E-class routers.
- Port names follow each vendor's style: Nokia `1/1/1`, Huawei `GigabitEthernet0/1/0`.
- All IP addresses are documentation ranges (RFC 5737) or private space; service IDs and names are made up.

> ⚠️ Syntax and output differ between software releases. Always check your platform's documentation before running commands on production equipment.

---

## Contents

1. [System & hardware](#1-system--hardware)
2. [Ports, interfaces & optics](#2-ports-interfaces--optics)
3. [Link aggregation & redundancy](#3-link-aggregation--redundancy)
4. [IP routing: IGP & BGP](#4-ip-routing-igp--bgp)
5. [MPLS: LDP, RSVP-TE & tunnels](#5-mpls-ldp-rsvp-te--tunnels)
6. [L2 services: Epipe / VLL and VPLS / VSI](#6-l2-services-epipe--vll-and-vpls--vsi)
7. [L3 services: VPRN / VPN instance](#7-l3-services-vprn--vpn-instance)
8. [Reachability tests](#8-reachability-tests)
9. [Logs & configuration](#9-logs--configuration)
10. [Terminology map](#10-terminology-map)

See also: **[Troubleshooting flows](troubleshooting.md)**

---

## 1. System & hardware

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| Software version | `show version` | `display version` |
| System summary | `show system information` | `display device` |
| Chassis, cards, MDAs | `show chassis` · `show card` · `show mda` | `display device` · `display elabel` |
| CPU usage | `show system cpu` | `display cpu-usage` |
| Memory | `show system memory-pools` | `display memory-usage` |

## 2. Ports, interfaces & optics

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| All ports, short | `show port` | `display interface brief` |
| One port in detail (errors, counters) | `show port 1/1/1 detail` | `display interface GigabitEthernet0/1/0` |
| Optical levels (Tx/Rx power) | `show port 1/1/1 optical` | `display transceiver interface GigabitEthernet0/1/0 verbose` |
| Live traffic rate | `monitor port 1/1/1 rate` | `display interface GigabitEthernet0/1/0` (input/output rate lines) |
| Clear counters | `clear port 1/1/1 statistics` | `reset counters interface GigabitEthernet0/1/0` |
| L3 interfaces | `show router interface` | `display ip interface brief` |
| ARP table | `show router arp` | `display arp all` |

**What to look for:** CRC / FCS errors, symbol errors and pause frames usually point at a physical-layer problem (dirty optic, bad patch cord, speed or duplex mismatch) long before the link actually goes down.

## 3. Link aggregation & redundancy

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| LAG / Eth-Trunk summary | `show lag` | `display eth-trunk` |
| One bundle in detail | `show lag 1 detail` | `display eth-trunk 1` |
| Multi-chassis LAG state | `show redundancy multi-chassis mc-lag peer 192.0.2.2` | `display e-trunk 1` |
| VRRP | `show router vrrp instance` | `display vrrp brief` |

Nokia **MC-LAG** and Huawei **E-Trunk** solve the same problem: a single LAG towards the customer, terminated on two different routers.

## 4. IP routing: IGP & BGP

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| Routing table | `show router route-table` | `display ip routing-table` |
| Routes inside a prefix | `show router route-table 10.0.0.0/24 longer` | `display ip routing-table 10.0.0.0 24 longer-match` |
| IS-IS adjacencies | `show router isis adjacency` | `display isis peer` |
| OSPF neighbours | `show router ospf neighbor` | `display ospf peer brief` |
| BGP summary | `show router bgp summary` | `display bgp peer` |
| One BGP neighbour | `show router bgp neighbor 192.0.2.2` | `display bgp peer 192.0.2.2 verbose` |
| VPNv4 BGP peers | `show router bgp summary family vpn-ipv4` | `display bgp vpnv4 all peer` |

## 5. MPLS: LDP, RSVP-TE & tunnels

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| LDP sessions | `show router ldp session` | `display mpls ldp session` |
| LDP label bindings | `show router ldp bindings` | `display mpls ldp lsp` |
| All LSPs | `show router tunnel-table` | `display mpls lsp` |
| RSVP-TE LSPs | `show router mpls lsp` | `display mpls te tunnel-interface` |
| RSVP sessions | `show router rsvp session` | `display mpls rsvp-te session` |
| Tunnels available to services | `show router tunnel-table` | `display tunnel-info all` |

**Mental model:** if the LDP session is up but the LSP to a remote PE is missing, check that the remote loopback is in the IGP first. LDP follows the IGP.

## 6. L2 services: Epipe / VLL and VPLS / VSI

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| List services | `show service service-using` | `display vsi` · `display mpls l2vc brief` |
| Point-to-point (Epipe / VLL) state | `show service id 100 base` | `display mpls l2vc` |
| Everything about one service | `show service id 100 all` | `display vsi name CUST-A verbose` |
| Customer attachment (SAP / AC) | `show service id 100 sap 1/1/1:100` | `display interface GigabitEthernet0/1/0.100` |
| Service tunnels (SDP / PW peers) | `show service sdp` | `display vsi name CUST-A peer-info` |
| VPLS MAC table | `show service id 100 fdb detail` | `display mac-address dynamic vsi CUST-A` |

**Common pitfalls:** a pseudowire that refuses to come up is very often an MTU mismatch between the two ends. In VPLS, MAC flapping is usually the first sign of a loop on the customer side.

## 7. L3 services: VPRN / VPN instance

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| List L3VPNs | `show service service-using vprn` | `display ip vpn-instance` |
| VRF routing table | `show router 200 route-table` | `display ip routing-table vpn-instance CUST-B` |
| VRF BGP (PE–CE) | `show router 200 bgp summary` | `display bgp vpnv4 vpn-instance CUST-B peer` |
| VRF ARP | `show router 200 arp` | `display arp vpn-instance CUST-B` |

**Common pitfall:** a route visible at one site but not another usually means a wrong or missing route-target.

## 8. Reachability tests

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| Ping | `ping 192.0.2.10` | `ping 192.0.2.10` |
| Ping inside a VRF | `ping 10.1.1.1 router 200` | `ping -vpn-instance CUST-B 10.1.1.1` |
| Traceroute inside a VRF | `traceroute 10.1.1.1 router 200` | `tracert -vpn-instance CUST-B 10.1.1.1` |
| Test an LDP LSP | `oam lsp-ping prefix 192.0.2.2/32` | `ping lsp ip 192.0.2.2 32` |
| Trace an LDP LSP | `oam lsp-trace prefix 192.0.2.2/32` | `tracert lsp ip 192.0.2.2 32` |
| Test a service tunnel (SDP) | `oam sdp-ping 10` | — (check `display mpls l2vc` / `display vsi ... peer-info`) |

## 9. Logs & configuration

| Task | Nokia SR OS | Huawei VRP |
|---|---|---|
| Recent logs | `show log log-id 99` | `display logbuffer` |
| Active alarms | usually monitored in NSP / SAM 5620 | `display alarm active` |
| Running configuration | `admin display-config` | `display current-configuration` |
| Save configuration | `admin save` | `save` |
| Saved (startup) configuration | `file type cf3:\config.cfg` | `display saved-configuration` |
| Filter the configuration | `admin display-config \| match "service" context all` | `display current-configuration \| include vsi` |

## 10. Terminology map

| Concept | Nokia | Huawei |
|---|---|---|
| Point-to-point L2 service | Epipe (VLL) | VLL / PWE3 (`mpls l2vc`) |
| Multipoint L2 service | VPLS | VSI |
| L3VPN | VPRN | VPN instance |
| Routed internet service | IES | Interface in the public routing table |
| Customer attachment point | SAP | AC (attachment circuit) |
| Service transport tunnel | SDP | PW over an LSP / tunnel policy |
| Multi-chassis LAG | MC-LAG | E-Trunk |
| Link aggregation | LAG | Eth-Trunk |
| Element / network management | SAM 5620 / NSP | U2000 / iMaster NCE |

---

## Contributing

Spotted a command that changed in your release, or want to add a section (EVPN, segment routing, multicast, QoS)? Pull requests are welcome. Please:

- keep every example free of real addresses, hostnames and customer data,
- mention the software release you tested on when it matters.

## License

[MIT](LICENSE) © Oğuzhan Canmar. Nokia and Huawei are trademarks of their respective owners; this project is not affiliated with either company.
