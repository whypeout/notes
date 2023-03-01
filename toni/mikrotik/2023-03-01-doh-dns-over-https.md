# Aktifkan DNS Over HTTPS (DOH) pada Mikrotik

source:
- [https://www.ramitan.com/tutorial/dns-over-https-mikrotik/](https://www.ramitan.com/tutorial/dns-over-https-mikrotik/)
- [https://labkom.co.id/mikrotik/cara-mengaktifkan-fitur-terbaru-dns-over-https-doh-di-mikrotik](https://labkom.co.id/mikrotik/cara-mengaktifkan-fitur-terbaru-dns-over-https-doh-di-mikrotik)
- [https://www.evotekno.com/tutorial/kumpulan-dns-over-https-yang-bisa-anda-pakai](https://www.evotekno.com/tutorial/kumpulan-dns-over-https-yang-bisa-anda-pakai)


IP->DNS->Static->Add

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
  - 



