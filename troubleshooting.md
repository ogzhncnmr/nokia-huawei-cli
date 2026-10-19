# Sorun giderme akışları

> 🇬🇧 [English version](troubleshooting.en.md)

Katman katman, kısa kontrol listeleri. Aşağıdan yukarıya doğru ilerleyin ve hata veren ilk katmanda durun.

Komutlar **Nokia** / **Huawei** sırasıyla verilmiştir. Tam liste için [README](README.md)'ye bakın.

---

## Pseudowire / Epipe ayakta değil

1. **Müşteri portu ve SAP / AC ayakta mı?**
   `show service id 100 sap 1/1/1:100` / `display interface GigabitEthernet0/1/0.100`
2. **Karşı PE'ye taşıma var mı?** Karşı loopback'e giden bir LSP var mı?
   `show router tunnel-table` / `display mpls lsp`
3. **LSP gerçekten trafik taşıyor mu?**
   `oam lsp-ping prefix 192.0.2.2/32` / `ping lsp ip 192.0.2.2 32`
4. **Sinyalleşme (targeted LDP) ayakta mı?**
   `show router ldp session` / `display mpls ldp session`
5. **Servis parametreleri iki uçta da aynı mı?** Servis MTU'su, VC ID ve VC tipi birebir aynı olmalı.
   `show service id 100 base` / `display mpls l2vc`

> Sahada en sık karşılaşılan kök neden: iki uç arasındaki **MTU uyumsuzluğu**.

---

## VPLS müşterisi aralıklı kayıp bildiriyor

1. **MAC flapping var mı?** Aynı MAC adresinin kısa aralıklarla farklı SAP'lerde ya da komşularda öğrenilmesi bir döngüye işaret eder.
   `show service id 100 fdb detail` / `display mac-address dynamic vsi CUST-A`
2. **Yayın fırtınası var mı?** Müşteriye bakan portlardaki trafik hızını kontrol edin.
   `monitor port 1/1/1 rate` / `display interface GigabitEthernet0/1/0`
3. **Fiziksel hata var mı?** CRC / FCS, symbol hataları, pause frame'ler.
   `show port 1/1/1 detail` / `display interface GigabitEthernet0/1/0`
4. **Optik seviyeler normal aralıkta mı?**
   `show port 1/1/1 optical` / `display transceiver interface GigabitEthernet0/1/0 verbose`

> Müşteri tarafındaki bir döngü, VPLS'in bütün lokasyonlarına yayılır. MAC limitleri ve yayın trafiği sınırlamaları bu yüzden önemlidir.

---

## Bir L3VPN lokasyonunda rota eksik

1. **Rota kaynak VRF'te var mı?**
   `show router 200 route-table` / `display ip routing-table vpn-instance CUST-B`
2. **PE–CE oturumu ayakta mı?**
   `show router 200 bgp summary` / `display bgp vpnv4 vpn-instance CUST-B peer`
3. **PE'ler arasındaki VPNv4 oturumu ayakta mı?**
   `show router bgp summary family vpn-ipv4` / `display bgp vpnv4 all peer`
4. **Route-target'lar eşleşiyor mu?** Kaynakta export edilen RT, hedefte import edilmelidir.
5. **Next-hop PE'ye taşıma LSP'si var mı?**
   `show router tunnel-table` / `display tunnel-info all`

---

## Link "ayakta" ama kullanıcılar şikâyetçi

1. Hata sayaçları artıyor mu? Sayaçları sıfırlayıp birkaç dakika izleyin.
   `clear port 1/1/1 statistics` / `reset counters interface GigabitEthernet0/1/0`
2. Rx gücü alıcının hassasiyet sınırına yakın mı?
3. LAG üyeleri dengeli mi, yoksa tek bir üye mi dolu?
   `show lag 1 detail` / `display eth-trunk 1`
4. Pause frame var mı? Aşağı yöndeki bir nokta tıkanıyor demektir.

---

*Örneklerde RFC 5737 belgeleme adresleri ve uydurma servis numaraları kullanılmıştır.*
