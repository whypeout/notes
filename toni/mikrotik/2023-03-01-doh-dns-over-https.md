# Aktifkan DNS Over HTTPS (DOH) pada Mikrotik

source:
- [https://www.ramitan.com/tutorial/dns-over-https-mikrotik/](https://www.ramitan.com/tutorial/dns-over-https-mikrotik/)
- [https://labkom.co.id/mikrotik/cara-mengaktifkan-fitur-terbaru-dns-over-https-doh-di-mikrotik](https://labkom.co.id/mikrotik/cara-mengaktifkan-fitur-terbaru-dns-over-https-doh-di-mikrotik)
- [https://www.evotekno.com/tutorial/kumpulan-dns-over-https-yang-bisa-anda-pakai](https://www.evotekno.com/tutorial/kumpulan-dns-over-https-yang-bisa-anda-pakai)

minimal requirements: MikroTik RouterOS v6.47

IP->DNS->Static->Add  

![image](https://user-images.githubusercontent.com/89820226/222049191-989bd277-e8c6-4b73-9165-ec6af4fdfeac.png)
![image](https://user-images.githubusercontent.com/89820226/222049217-2ccfe5d0-5f47-4519-850e-e15b45f9fcfb.png)

klik disini untuk check dns [https://1.1.1.1/help](https://1.1.1.1/help)

name: cloudflare-dns.com  
address: 104.16.249.249, 104.16.248.249  
use-doh-server: https://cloudflare-dns.com/dns-query  

name: dns.google  
address: 8.8.8.8, 8.8.4.4  
use-doh-server: https://dns.google/dns-query  

list:
- aaflalo.me (US)
  - https://dns-nyc.aaflalo.me/dns-query
- adGuard support dnssec & adblock
  - https://dns.adguard.com/dns-query
  - https://dns-family.adguard.com/dns-query
  - https://dns-unfiltered.adguard.com/dns-query
- ahaDNS doh dot, blok malware tracker virus telemetry. support dnssec tls1.3 opensource
  - https://doh.nl.ahadns.net/dns-query
  -  https://doh.in.ahadns.net/dns-query
- alibaba dns, fitur: doh dot dns-json-api (best-china). dnssec adblock
  - https://dns.alidns.com/dns-query
- alekberg: doh, no logging filter, dnssec
  - https://doh.in.ahadns.net/dns-query spain
  - https://dnsnl.alekberg.net/dns-query holland
  - https://dnsse.alekberg.net/dns-query swedia
- association 42l: dnssec, no log query, doh-proxy edgedns caching
  - https://doh.42l.fr/dns-query
- blahdns.com: go, knot-resolver, unbound-dnssec no-ecs no-logs adblock
  - https://doh-sg.blahdns.com/dns-query
  - https://doh-de.blahdns.com/dns-query
  - https://doh-jp.blahdns.com/dns-query
  - https://doh-fi.blahdns.com/dns-query
- bravedns: dns cepat aman private transparent configured. no ecs. cname.
  - https://free.bravedns.com/dns-query malware-adblocking
  - https://bravedns.com/configure endpoint-config
- cloudflare
  - https://mozilla.cloudflare-dns.com/dns-query mozilla
  - 1dot1dot1dot1.cloudflare-dns.com 1.1.1.1 android
  - https://security.cloudflare-dns.com/dns-query malware
  - https://family.cloudflare-dns.com/dns-query adult & malware
  - https://dns64.cloudflare-dns.com/dns-query dns64
  - https://dns.cloudflare.com/dns-query
- google
  - https://dns.google/dns-query
  - https://8888.google/dns-query
- quad9
  - https://dns.quad9.net/dns-query recommended
  - https://dns9.quad9.net/dns-query secure
  - https://dns10.quad9.net/dns-query un-secure
  - https://dns11.quad9.net/dns-query secure no-ecs
- opendns
  - https://doh.opendns.com/dns-query
  - https://doh.familyshield.opendns.com/dns-query
  - https://asia.dnscepat.id/dns-query
  - https://doh.cleanbrowsing.org/doh/family-filter/
  - https://doh.cleanbrowsing.org/doh/adult-filter/
  - https://doh.cleanbrowsing.org/doh/security-filter/


