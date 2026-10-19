# Troubleshooting flows

> 🇹🇷 [Türkçe sürüm](troubleshooting.md)

Short, layer-by-layer checklists. Work from the bottom up and stop at the first layer that fails.

Commands are shown as **Nokia** / **Huawei**. See the [README](README.en.md) for the full reference.

---

## A pseudowire / Epipe is down

1. **Customer port and SAP / AC up?**
   `show service id 100 sap 1/1/1:100` / `display interface GigabitEthernet0/1/0.100`
2. **Transport to the remote PE?** Is there an LSP to the remote loopback?
   `show router tunnel-table` / `display mpls lsp`
3. **LSP actually forwards?**
   `oam lsp-ping prefix 192.0.2.2/32` / `ping lsp ip 192.0.2.2 32`
4. **Signalling (targeted LDP) up?**
   `show router ldp session` / `display mpls ldp session`
5. **Service parameters match on both ends?** Service MTU, VC ID and VC type must be identical.
   `show service id 100 base` / `display mpls l2vc`

> Most common root cause in practice: **MTU mismatch** between the two ends.

---

## A VPLS customer reports intermittent loss

1. **MAC flapping?** The same MAC learned on different SAPs / peers in quick succession means a loop.
   `show service id 100 fdb detail` / `display mac-address dynamic vsi CUST-A`
2. **Broadcast storm?** Check the rate on the customer-facing ports.
   `monitor port 1/1/1 rate` / `display interface GigabitEthernet0/1/0`
3. **Physical errors?** CRC / FCS, symbol errors, pause frames.
   `show port 1/1/1 detail` / `display interface GigabitEthernet0/1/0`
4. **Optics within range?**
   `show port 1/1/1 optical` / `display transceiver interface GigabitEthernet0/1/0 verbose`

> A loop on the customer side spreads to every site of the VPLS. MAC limits and broadcast rate limits are your friends.

---

## A route is missing at one L3VPN site

1. **Route in the source VRF?**
   `show router 200 route-table` / `display ip routing-table vpn-instance CUST-B`
2. **PE–CE session up?**
   `show router 200 bgp summary` / `display bgp vpnv4 vpn-instance CUST-B peer`
3. **VPNv4 session between PEs up?**
   `show router bgp summary family vpn-ipv4` / `display bgp vpnv4 all peer`
4. **Route-targets match?** Export RT on the source must be imported on the destination.
5. **Transport LSP to the next-hop PE?**
   `show router tunnel-table` / `display tunnel-info all`

---

## A link is "up" but users complain

1. Error counters climbing? Clear them and watch for a few minutes.
   `clear port 1/1/1 statistics` / `reset counters interface GigabitEthernet0/1/0`
2. Rx power close to the receiver sensitivity limit?
3. LAG members balanced, or is one member saturated?
   `show lag 1 detail` / `display eth-trunk 1`
4. Pause frames present? Something downstream is congested.

---

*Examples use RFC 5737 documentation addresses and made-up service IDs.*
