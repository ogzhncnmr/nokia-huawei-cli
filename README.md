# nokia-huawei-cli

**IP/MPLS işletimi ve sorun giderme için Nokia SR OS ↔ Huawei VRP komutlarını yan yana gösteren başvuru kaynağı.**

> 🇬🇧 [English version](README.en.md)

Birden fazla üreticinin bir arada çalıştığı operatör ağlarında, iki farklı CLI arasında sürekli kafanızda çeviri yaparsınız. Bu repo, her gün kullandığınız komutlar için o çeviriyi sizin yerinize yapar.

- **Nokia** örnekleri **klasik CLI** ile yazılmıştır (7750 SR / 7450 ESS / 7950 XRS üzerinde SR OS).
- **Huawei** örnekleri NE40E sınıfı yönlendiricilerdeki **VRP** içindir.
- Port adları her üreticinin kendi yazımına göredir: Nokia `1/1/1`, Huawei `GigabitEthernet0/1/0`.
- Tüm IP adresleri belgeleme aralıklarından (RFC 5737) ya da özel adres alanından seçilmiştir; servis numaraları ve adları uydurmadır.

> ⚠️ Komut sözdizimi ve çıktılar yazılım sürümüne göre değişebilir. Canlı cihazlarda çalıştırmadan önce mutlaka kendi platformunuzun dokümantasyonunu kontrol edin.

---

## İçindekiler

1. [Sistem ve donanım](#1-sistem-ve-donanım)
2. [Portlar, arayüzler ve optikler](#2-portlar-arayüzler-ve-optikler)
3. [Bağlantı birleştirme ve yedeklilik](#3-bağlantı-birleştirme-ve-yedeklilik)
4. [IP yönlendirme: IGP ve BGP](#4-ip-yönlendirme-igp-ve-bgp)
5. [MPLS: LDP, RSVP-TE ve tüneller](#5-mpls-ldp-rsvp-te-ve-tüneller)
6. [L2 servisler: Epipe (VLL) ve VPLS (VSI)](#6-l2-servisler-epipe-vll-ve-vpls-vsi)
7. [L3 servisler: VPRN ve VPN instance](#7-l3-servisler-vprn-ve-vpn-instance)
8. [Erişilebilirlik testleri](#8-erişilebilirlik-testleri)
9. [Loglar ve konfigürasyon](#9-loglar-ve-konfigürasyon)
10. [Terim eşleştirme tablosu](#10-terim-eşleştirme-tablosu)

Ayrıca bakınız: **[Sorun giderme akışları](troubleshooting.md)**

---

## 1. Sistem ve donanım

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Yazılım sürümü | `show version` | `display version` |
| Sistem özeti | `show system information` | `display device` |
| Şasi, kartlar, MDA'lar | `show chassis` · `show card` · `show mda` | `display device` · `display elabel` |
| CPU kullanımı | `show system cpu` | `display cpu-usage` |
| Bellek | `show system memory-pools` | `display memory-usage` |

## 2. Portlar, arayüzler ve optikler

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Tüm portlar, kısa liste | `show port` | `display interface brief` |
| Tek port, ayrıntılı (hatalar, sayaçlar) | `show port 1/1/1 detail` | `display interface GigabitEthernet0/1/0` |
| Optik seviyeler (Tx/Rx gücü) | `show port 1/1/1 optical` | `display transceiver interface GigabitEthernet0/1/0 verbose` |
| Anlık trafik hızı | `monitor port 1/1/1 rate` | `display interface GigabitEthernet0/1/0` (input/output rate satırları) |
| Sayaçları sıfırlama | `clear port 1/1/1 statistics` | `reset counters interface GigabitEthernet0/1/0` |
| L3 arayüzler | `show router interface` | `display ip interface brief` |
| ARP tablosu | `show router arp` | `display arp all` |

**Nelere bakmalı:** CRC / FCS hataları, symbol hataları ve pause frame'ler, link düşmeden çok önce fiziksel katmanda bir sorun olduğunu gösterir: kirli optik, kötü patch kablo, hız ya da duplex uyumsuzluğu.

## 3. Bağlantı birleştirme ve yedeklilik

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| LAG / Eth-Trunk özeti | `show lag` | `display eth-trunk` |
| Tek bir bundle, ayrıntılı | `show lag 1 detail` | `display eth-trunk 1` |
| Çoklu şasi LAG durumu | `show redundancy multi-chassis mc-lag peer 192.0.2.2` | `display e-trunk 1` |
| VRRP | `show router vrrp instance` | `display vrrp brief` |

Nokia **MC-LAG** ve Huawei **E-Trunk** aynı sorunu çözer: müşteriye doğru tek bir LAG, iki farklı yönlendiricide sonlandırılır.

## 4. IP yönlendirme: IGP ve BGP

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Yönlendirme tablosu | `show router route-table` | `display ip routing-table` |
| Bir önekin içindeki rotalar | `show router route-table 10.0.0.0/24 longer` | `display ip routing-table 10.0.0.0 24 longer-match` |
| IS-IS komşulukları | `show router isis adjacency` | `display isis peer` |
| OSPF komşuları | `show router ospf neighbor` | `display ospf peer brief` |
| BGP özeti | `show router bgp summary` | `display bgp peer` |
| Tek bir BGP komşusu | `show router bgp neighbor 192.0.2.2` | `display bgp peer 192.0.2.2 verbose` |
| VPNv4 BGP komşuları | `show router bgp summary family vpn-ipv4` | `display bgp vpnv4 all peer` |

## 5. MPLS: LDP, RSVP-TE ve tüneller

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| LDP oturumları | `show router ldp session` | `display mpls ldp session` |
| LDP etiket eşleşmeleri | `show router ldp bindings` | `display mpls ldp lsp` |
| Tüm LSP'ler | `show router tunnel-table` | `display mpls lsp` |
| RSVP-TE LSP'ler | `show router mpls lsp` | `display mpls te tunnel-interface` |
| RSVP oturumları | `show router rsvp session` | `display mpls rsvp-te session` |
| Servislerin kullanabileceği tüneller | `show router tunnel-table` | `display tunnel-info all` |

**Akılda tutun:** LDP oturumu ayakta ama karşı PE'ye giden LSP yoksa, önce karşı loopback adresinin IGP'de olup olmadığına bakın. LDP, IGP'yi takip eder.

## 6. L2 servisler: Epipe (VLL) ve VPLS (VSI)

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Servisleri listeleme | `show service service-using` | `display vsi` · `display mpls l2vc brief` |
| Noktadan noktaya (Epipe / VLL) durumu | `show service id 100 base` | `display mpls l2vc` |
| Tek servisin her şeyi | `show service id 100 all` | `display vsi name CUST-A verbose` |
| Müşteri bağlantı noktası (SAP / AC) | `show service id 100 sap 1/1/1:100` | `display interface GigabitEthernet0/1/0.100` |
| Servis tünelleri (SDP / PW komşuları) | `show service sdp` | `display vsi name CUST-A peer-info` |
| VPLS MAC tablosu | `show service id 100 fdb detail` | `display mac-address dynamic vsi CUST-A` |

**Sık karşılaşılan tuzaklar:** Ayağa kalkmayan bir pseudowire'ın en sık nedeni iki uç arasındaki MTU uyumsuzluğudur. VPLS'te MAC flapping (aynı MAC'in sürekli farklı yerlerde görülmesi) genellikle müşteri tarafındaki bir döngünün ilk işaretidir.

## 7. L3 servisler: VPRN ve VPN instance

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| L3VPN'leri listeleme | `show service service-using vprn` | `display ip vpn-instance` |
| VRF yönlendirme tablosu | `show router 200 route-table` | `display ip routing-table vpn-instance CUST-B` |
| VRF içi BGP (PE–CE) | `show router 200 bgp summary` | `display bgp vpnv4 vpn-instance CUST-B peer` |
| VRF ARP tablosu | `show router 200 arp` | `display arp vpn-instance CUST-B` |

**Sık karşılaşılan tuzak:** Bir rota bir lokasyonda görünüp diğerinde görünmüyorsa, sebep çoğu zaman yanlış ya da eksik bir route-target'tır.

## 8. Erişilebilirlik testleri

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Ping | `ping 192.0.2.10` | `ping 192.0.2.10` |
| VRF içinden ping | `ping 10.1.1.1 router 200` | `ping -vpn-instance CUST-B 10.1.1.1` |
| VRF içinden traceroute | `traceroute 10.1.1.1 router 200` | `tracert -vpn-instance CUST-B 10.1.1.1` |
| LDP LSP testi | `oam lsp-ping prefix 192.0.2.2/32` | `ping lsp ip 192.0.2.2 32` |
| LDP LSP izleme | `oam lsp-trace prefix 192.0.2.2/32` | `tracert lsp ip 192.0.2.2 32` |
| Servis tüneli (SDP) testi | `oam sdp-ping 10` | — (`display mpls l2vc` / `display vsi ... peer-info` ile kontrol edin) |

## 9. Loglar ve konfigürasyon

| İşlem | Nokia SR OS | Huawei VRP |
|---|---|---|
| Son loglar | `show log log-id 99` | `display logbuffer` |
| Aktif alarmlar | genellikle NSP / SAM 5620 üzerinden izlenir | `display alarm active` |
| Çalışan konfigürasyon | `admin display-config` | `display current-configuration` |
| Konfigürasyonu kaydetme | `admin save` | `save` |
| Kayıtlı (açılış) konfigürasyonu | `file type cf3:\config.cfg` | `display saved-configuration` |
| Konfigürasyonu filtreleme | `admin display-config \| match "service" context all` | `display current-configuration \| include vsi` |

## 10. Terim eşleştirme tablosu

| Kavram | Nokia | Huawei |
|---|---|---|
| Noktadan noktaya L2 servis | Epipe (VLL) | VLL / PWE3 (`mpls l2vc`) |
| Çok noktalı L2 servis | VPLS | VSI |
| L3VPN | VPRN | VPN instance |
| Yönlendirilmiş internet servisi | IES | Genel yönlendirme tablosundaki arayüz |
| Müşteri bağlantı noktası | SAP | AC (attachment circuit) |
| Servis taşıma tüneli | SDP | LSP üzerinde PW / tunnel policy |
| Çoklu şasi LAG | MC-LAG | E-Trunk |
| Bağlantı birleştirme | LAG | Eth-Trunk |
| Eleman / ağ yönetimi | SAM 5620 / NSP | U2000 / iMaster NCE |

---

## Katkıda bulunma

Kendi sürümünüzde değişmiş bir komut mu gördünüz, ya da yeni bir bölüm (EVPN, segment routing, multicast, QoS) eklemek mi istiyorsunuz? Pull request'ler memnuniyetle karşılanır. Lütfen:

- örneklerde gerçek adres, hostname ya da müşteri verisi kullanmayın,
- önemliyse test ettiğiniz yazılım sürümünü belirtin.

## Lisans

[MIT](LICENSE) © Oğuzhan Canmar. Nokia ve Huawei, sahiplerinin ticari markalarıdır; bu proje iki şirketle de bağlantılı değildir.
