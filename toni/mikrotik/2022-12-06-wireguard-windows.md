# Remote Tunnel VPN dg Wireguard pada Windows

sources:
- [https://golb.hplar.ch/2019/07/wireguard-windows.html](https://golb.hplar.ch/2019/07/wireguard-windows.html)


![image](https://user-images.githubusercontent.com/89820226/205838335-6dde74d2-dc1a-4547-94c3-6bb3681e780b.png)

download wireguard for windows [https://www.wireguard.com/install/](https://www.wireguard.com/install/)

![image](https://user-images.githubusercontent.com/89820226/205875204-03dfcd37-d493-4ad0-8780-684031119941.png)

konfigurasi wiregurad vpn tunnel

## Windows 

![image](https://user-images.githubusercontent.com/89820226/205875358-f9e2ea56-efba-41d5-9012-52e90d80b4ed.png)

> WireGuard GUI -> Add Tunnel -> Add empty Tunnel ...

![image](https://user-images.githubusercontent.com/89820226/205875486-4d3e2ab8-80c3-45b1-a5d8-fc3050f25856.png)

```
[Interface]
PrivateKey = 6I79zNsp26O2gCYkScEXdypB2UZ8IbNv2pV36QstKlo=
# local wireguard interface ip
Address = 192.168.2.2/32
# wireguard interface ip di server
DNS = 192.168.2.1

[Peer]
# public key server/peer
PublicKey = uZik78EWgYCLQRMdG6k6QK0mzHFqfr4uhOEjPyXe5WE=
# enable default gw/tunnel vpn
AllowedIPs = 0.0.0.0/0
# ip public server/peer/vps
Endpoint = 35.174.118.17:54321
```

## Server

tambahkan `[peer]` di konfigurasi wireguard `/etc/wireguard/wg0.conf`

```
[Peer]
# public key milik windows client (utk otentikasi)
PublicKey = rbkuZ+3SyPtT/QLZhFhiTo555ekSCJRsHf3jJb5kdkI=
# ip client. jika tidak cocok, tidak dapat bertukar data. meskipun status sudah suskes konek
AllowedIPs = 192.168.2.2/32
```

![image](https://user-images.githubusercontent.com/89820226/205876287-6c172cb1-31f4-4dfa-8809-a3aa4e3c8978.png)

## Activate

utk menghubungkan wireguard client ke server

![image](https://user-images.githubusercontent.com/89820226/205876490-d170fb42-ece5-4a58-9b54-3cb26b885480.png)
![image](https://user-images.githubusercontent.com/89820226/205876506-b7b7db59-9406-4e96-bf8a-85ef41d2b904.png)

> Deactive utk mematikan vpn

## Check VPN Connection

buka [https://www.whatismyip.com/](https://www.whatismyip.com/) atau [https://icanhazip.com/](https://icanhazip.com/) atau [https://ipinfo.io](https://ipinfo.io)

## Tambahkan lebih banyak client

setiap kali menambahkan `[peer]` client kita harus me-restart wireguard:
```
wg-quick down wg0 && wg-quick up wg0
```



