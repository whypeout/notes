# Membuat Docker Container di Local Network dg MACVLAN dan di web proxy Traefik

sources:
- [https://dev.to/fredlab/make-docker-containers-available-both-on-y]our-local-network-with-macvlan-and-on-the-web-with-traefik-2hj1](https://dev.to/fredlab/make-docker-containers-available-both-on-your-local-network-with-macvlan-and-on-the-web-with-traefik-2hj1)
- [https://stackoverflow.com/questions/61831255/how-to-create-a-docker-macvlan-with-user-defined-ip-and-mac-address-using-compos](https://stackoverflow.com/questions/61831255/how-to-create-a-docker-macvlan-with-user-defined-ip-and-mac-address-using-compos)

media server seperti emby dan jellyfin menggunakan DLNA server yg tidak bisa di akses lewat docker port forward,
DLNA client seperti PS4 tidak bisa mengakses DLNA server pada container karena tidak berada di network yg sama.

## network_mode: host

```
# docker-compose.yml official jellyfin
version: '3'
services:
  jellyfin:
    image: jellyfin/jellyfin
    user: 1000:1000
    network_mode: "host"
    restart: "unless-stopped"
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
```

**network_mode: host** akan mem-bind semua port ke host machine ke container port. DLNA server port 1900 pada container akan di bind di host.

pada docker-compose, kita tidak bisa menggunakan **network_mode** dan **networks** bersamaan. kita cuma bisa pakai salah satu.
paling mudah jika kita tidak menggunakan **network** sama sekali.
tapi jika kita punya banyak container dibelakang reverse proxy, dg Traefik, kita perlu **network** utk me-route-kan http req ke container.

## MACVLAN driver

macvlan mengijinkan container mempunyai mac address, dan container akan terlihat di LAN sbg single host/device di LAN.

### Check driver macvlan

```
lsmod | grep macv
```

### List ethernet device

```
ifconfig -a
# atau
ip a

# eth0 enp3s0
```

### Emby & Jellyfin

```
# nano docker-compose.yml
version: '2.4'
services:
  emby:
    image: emby/embyserver:latest
    container_name: emby
    restart: unless-stopped
    volumes:
      - ./emby/config:/config
      - /media/share:/mnt/share
    devices:
      - /dev/dri:/dev/dri
    environment:
      - UID=1000
      - GID=1000
    mac_address: 00:11:32:1a:2b:30
    netrowks:
      lan:
        ipv4_address: 192.168.8.2
        
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: 1000:1000
    restart: unless-stopped
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /media/share:/media
    mac_address: 00:11:32:1a:2b:31
    networks:
      lan:
        ipv4_address: 192.168.8.3

networks:
  lan:
    name: lan
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      config:
        - subnet: 192.168.8.0/24
          gateway: 192.168.8.254
          ip_range: 192.168.8.126/25
```

dirumah, mungkin kita tidak perlu emby dan jellyfin bersamaan. cukup remove salah satu sesuai keperluan.

jangan lupa utk membuat volume di lokal disk utk menyimpan data container.

kedua container menggunakan UID dan GID dari user di host machine

> note: macvlan network hanya berjalan pada docker-compose version "2.*"

kita buat network **lan**, jadi setiap container yg menggunakan network ini akan menggunakan macvlan driver ke **parent** interface tertentu (eth0 ethernet).

didalam **ipam** dan **config** kita bisa beri beberapa options. seperti membuat container berada di **subnet** yg sama dg perangkat lokal kita. dg **gateway** dan **ip_range** sehingga tidak konflik dg dhcp & perangkat lokal lain.

kita juga bisa tambahkan **mac_address** utk kemudian kita daftarkan di dhcp server kita utk mengatur ip dari server dhcp.
atau kita bisa atur manual ip address di docker-compose.

## Start container

```
docker-compose up -d

docker-compose ps

docker network ls

docker network inspect lan

docker container inspect emby jellyfin
```

interface router -> ip arp

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--CLey5882--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://i.ibb.co/58JNxND/Capture-d-cran-du-2020-08-01-17-19-43.png)
![img](https://res.cloudinary.com/practicaldev/image/fetch/s--sTQUZr5U--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://i.ibb.co/4t1WPvq/DSC-0072.jpg)

# Configure Traefik

> kita perlu nama domain

kita perlu network utk traefik memanage setiap container.

```
docker network create web
```

kita buat config `traefik.toml`. utk akses web interface, kita bisa sesuaikan username & password, juga domain name.

```
# nano traefik.toml
debug = true

logLevel = "DEBUG"
defaultEntryPoints = ["https","http"]

[api]
  entryPoint = "traefik"
  dashboard = true
  
  [entryPoints]
    [entryPoints.http]
    address = ":80"
      [entryPoints.http.redirect]
      entryPoint = "https"
    [entryPoints.https]
    address = ":443"
    [entryPoints.https.tls]
    [entryPoints.traefik]
      address = ":8080"
      [entryPoints.traefik.auth]
        [entryPoints.traefik.auth.basic]
        users = ["username:password"]
        
[retry]

[docker]
endpoint = "unix://var/run/docker.sock"
domain = "www.domain.lan"
watch = true
exposedByDefault = false
usebindportip = true

[acme]
email = "foo@bar.com"
storage = "/etc/traefik/acme.json"
entryPoint = "https"

onHostRule = true
[acme.httpChallenge]
entryPoint = "http"
```

```
# buat acme.json & rubah permisionnya:
touch acme.json
chmod 0660 acme.json
```

kita tambahkan traefik ke docker-compose.yml

```
...
  traefik:
    image: traefik:alpine
    container_name: traefik
    ports:
      - 80:80
      - 443:443
    restart: always
    volumes:
      - ./traefik.toml:/etc/traefik/traefik.toml
      - ./acme.json:/etc/traefik/acme.json
      - /var/run/docker.sock:/var/run/docker.sock
    labels:
      - "treafik.enable=true"
      - "traefik.docker.network=web"
      - "traefik.port=8080"
      - "traefik.backend=traefik"
      - "traefik.frontend.rule=Host:traefik.domain.lan"
    networks:
      - web
...
  jellyfin:
    networks:
      web:
    labels:
      - "traefik.docker.network=web"
      - "traefik.port=8096"
      - "traefik.enable=true"
      - "traefik.backend=jellyfin"
      - "traefik.frontend.rule=Host:jellyfin.domain.lan"
...
  emby:
    networks:
      web:
    labels:
      - "traefik.docker.network=web"
      - "traefik.port=8096"
      - "traefik.enable=true"
      - "traefik.backend=emby"
      - "traefik.frontend.rule=Host:emby.domain.lan"
...
networks:
  web:
    external: true
...
```

kita buat ulang container nya:

```
docker-compose up -d
# atau
docker-compose up -d --force-recreate
```












































