# Tunnel Wireguard over TCP with UDPTUNNEL

sources: 
- [https://gist.github.com/insdavm/90cbeffe76ba4a51251d83af604adf94](https://gist.github.com/insdavm/90cbeffe76ba4a51251d83af604adf94)
- [https://manpages.ubuntu.com/manpages/focal/man1/udptunnel.1.html](https://manpages.ubuntu.com/manpages/focal/man1/udptunnel.1.html)
- alternative udptunnel [https://github.com/wangyu-/udp2raw](https://github.com/wangyu-/udp2raw)

udptunnel membuat tunnel UDP lewat koneksi TCP bi-directional.

## Server

```
# udptunnel -s 443 127.0.0.1/51820
```

> pastikan port 443 dibuka pada firewall server

> ubah listen port pada wireguard server ke 51820

## Client

```
# udptunnel -c [ip-public-server]/443 127.0.0.1 50001
```

> ubah config wireguard client utk endpoint addr nya ke 127.0.0.1 dan port 50001
