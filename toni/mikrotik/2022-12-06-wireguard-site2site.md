# WireGuard :: Site to Site VPN :: RouterOS v7

sources:
- [https://systemzone.net/wireguard-site-to-site-vpn-between-mikrotik-routeros-7/](https://systemzone.net/wireguard-site-to-site-vpn-between-mikrotik-routeros-7/)

Wireguard bisa digunakan dg Mode Client-Server VPN dan Site-to-Site VPN.
Wireguard menggunakan UDP utk Transmit Data.
Salah satu endpoint/router harus memiliki IP Public sbg Server, client/endpoint lain bisa menjadi Peer ke router/client lain di jaringan lokal.
Wireguard menggunakan Public Key utk Otentikasi antar Peer/Client-Server.
Gunakan AllowedIPs sebaik mungkin. utk site-to-site, misal network: 192.168.0.0/16, atau utk Internet Gateway 0.0.0.0/0

## Diagram Site-to-Site

![image](https://user-images.githubusercontent.com/89820226/205818283-e5b2b785-4935-43b0-9a51-af6c7d193b27.png)

## Enable Wireguard di Mikrotik Routeros

Wireguard -> Interface -> Add +

- `name: wireguard1` interface name
- `private key` let it be
- `Listen Port` for IP Public Router as Server / endpoint dimana semua peer akan terkoneksi
- `Public Key` copas, kita perlu ini utk peer/client

![image](https://user-images.githubusercontent.com/89820226/205818687-8b164436-a16a-405c-990b-f5bece13d6c3.png)

## Tambahkan IP address pada Wireguard Virtual Interface

IP -> Address -> Add

- `interface: wireguard1` nama interface wireguard
- `address: 192.168.250.0/24` network utk vpn wireguard

> R1 250.1 <==> 250.2 R2

## Tambahkan Peers pada Masing2 endpoint/site

site A : R1 Router utama (dg ip public)
site X : R2 Router di site lain (bisa tanpa ip public)

![image](https://user-images.githubusercontent.com/89820226/205821704-77e518aa-0e8f-47ba-bdc7-3d173649100f.png)
![image](https://user-images.githubusercontent.com/89820226/205821822-a22fdef4-fff2-4d1a-b1b1-5ba81d571f46.png)

- `interface: wireguard1` yg sudah dibuat tadi
- `public key` copas dari peer. misal utk R1 peer public key berisi public key dari R2,dan sebaliknya.
- `endpoint` IP public wireguard R1 (dan R2 jika ada ip public statis)
- `endpoint port` port pada peer. misal peer ke R1 menggunakan port 12321
- `Allowed Address: 192.168.250.0/24, 192.168.25.0/24, 192.168.2.0/24` => 250 network wireguard1, 25 network lokal site A, 2 network lokal site B
- `Persistent Keepalive 10s` range waktu router utk mengecek status tunnel agar tetap terkoneksi, atw mengupdate status ke diskonek
 
> Allowed IP: 0.0.0.0/0 utk membuat Remote Tunnel VPN dg mengarahkan semua paket data melalui wireguard.

## Static Routing antara R1 dan R2

R1>ip routes add
- `dst-address: 192.168.2.0/24` network R2 router/cabang
- `gateway: 192.168.250.1` ip interface R1 wireguard1

R2>ip routes add
- `dst-address: 192.168.25.0/24` network R1 router/pusat
- `gateway: 192.168.250.1` ip interface R2 wireguard1

![image](https://user-images.githubusercontent.com/89820226/205838216-2c338f69-0981-4b31-b922-634b588e7dfa.png)








