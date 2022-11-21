# Docker Reverse Proxy menggunakan Traefik

sources:
- [https://accesto.com/blog/docker-reverse-proxy-using-traefik/](https://accesto.com/blog/docker-reverse-proxy-using-traefik/)

## Kenapa kita perlu Reverse Proxy Server ?

kita perlu reverse proxy utk docker/compose config, sbb:
- routing inbound trafik ke container yg tepat di multi-container env. misal route req ke web server
- terminate SSL (Lets Encrypt)
- menjadi load balancing utk multiple backend server env
- basic auth
- IP whitelist/blacklist

berikut gambaran reverse proxy didepan app dan terminate SSL, dan me-route client req ke backend web server.

![img](https://accesto.com/blog/static/89ce32385de2566440887970ae957a82/3c492/reverse-proxy-example.png)

dg reverse proxy, kita bisa split incoming trafik ke beberapa server, semua bekerja di dalam internal network dan terbuka di sebuah single ip public.

reverse proxy yg baik dpt melindungi dari hacker, misal filter HTTP request (log4j vulnerability).

## Reverse Proxy di Docker

biasanya kita pakai NGINX sbg reverse proxy. dikonfigurasi dg labels, sbg implementasi termudah.

## Traefik

built dg 4 konsep utama: entrypoint, router, middleware dan service.
listen di entrypoint tertentu (ports, TCP/HTTP port 80), incoming req is matched to a route, setiap req bisa melewati middleware (beberapa). spt path rewrite, compression dll.
akhirnya mencapai service sesuai konfigurasi utk route yg tepat. service biasanya map ke app web server.

![img](https://accesto.com/blog/static/61da49d156c5a646363d252c12f65ccf/3c492/traefik-components-overview.png)

## Basic Setup

trafik support beberapa config provider, termasuk file atau HTTP endpoint. disini kita gunakan Docker.
menggunakan labels spt pada nginx-proxy, dg beberapa konfig lebih lengkap.

```yaml
version: '3.3'
services:
  traefik:
    image: "traefik:v2.6"
    command:
      #- "--log.level=DEBUG"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

- `api.insecure=true` enable traefik dashboard. berguna utk debugging di env development. utk keamanan lebih baik disable pada env production.
- `providers.docker=true` enable docker config discovery
- `providers.docker.exposedbydefault=false` jangan expose docker service secara default
- `entrypoints.web.address=80` buat entrypoint `web` listen di port `80`

kita expose port 80 utk akses `web` entrypoint, dan port `8080` sbg default dashboard port. kita juga konekkan volume `docker.sock` jdi traefik bisa berkomunikasi dg docker daemon (mengambil informasi container running)

```
whoami:
  image: "traefik/whoami"
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
    - "traefik.http.routers.whoami.entrypoints=web"
```

- `traefik.enable=true` memberitahu traefik utk expose service ini
- `traefik.http.routers.whoami.rule=Host("whoami.localhost")` rule utk match req ke service ini. kita pakai router nama `whoami`, kita pakai matcher Host dg hostname `whoami.localhost`. ada matcher lain spt path, kita akan bahas nanti.
- `traefik.http.routers.whoami.entrypoints=web` entrypoint yg digunakan oleh service `whoami`

kita bisa buat multiple routers utk setiap container, cukup rubah `router name` (unique)

```
docker-compose up -d
```

> kita rubah service labels whoami `Host("localhost")`

![img](https://accesto.com/blog/static/7eb6ab185fc62eeab514354aea010c30/9cab2/proxied-service.png)
![dashboard](https://accesto.com/blog/static/6c05e8f28f0288c8489d335637bc4035/e6c84/traefik-dashboard.png)
![dashboard](https://accesto.com/blog/static/61f7fec9acefbde084b6c72d61484f11/8cdda/traefik-route-view.png)

## Load Balancing

dg konfig diatas, kita juga sudah konfig utk loadbalance. jika kita scale service `whoami` di docker-compose:

```
...
whoami:
  image: "traefik/whoami"
  scale: 5
  labels:
...
```

di dashboard traefik akan mengkoneksikan semua containers ke service:

![servers](https://accesto.com/blog/static/0d09bc4721e1cc923b988eaf158e90d0/3c492/traefik-load-balancer.png)

dan membagi trafik secara rata. sbgmana sebuah loadbalancer.

## Multiple service dan path matching

sperti yg kita duga, menambah service ke reverse proxy sangat mudah. kita bisa buat domain berbeda - dg menyeting `Host("localhost")`.
tapi kita juga bisa menggunakan route path utk service yg berbeda. contoh:

```
whoami2:
  image: "nginxdemos/hello"
  scale: 1
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.whoami2.rule=Host(`localhost`) && Path(`/whoami2`)"
    - "traefik.http.routers.whoami2.entrypoints=web"
```

![hasil](https://accesto.com/blog/static/81b45ffd40fc1f8a9cdc84274c0da0f4/3c492/traefik-path-routing.png)

- kita pakai image berbeda utk serve service HTTP lain
- kita buat dg nama baru, `whoami2`
- kita tambah rule route `&& Path("/whoami2")`. jadi harus match hostname `localhost` dan match path `whoami2`, jika hanya perlu path prefix cukup pakai `PathPrefix("/whoami2")`

## SSL Encryption

kita kadang perlu thin layer antara app dan client, utk mempermudah pembuatan & update certificate. letsencrypt cukup bagus, tapi dg task utk expose docker service dg cepat perlu beberapa trik.

traefik punya resolver certificate bawaan.kita hanya perlu menyebutkan resolver/provider yg akan digunakan utk menghandle semua cert yg diperlukan. jadi jika kita punya instance traefik yg sudah running dan menambahkan domain `accessto.com` ke salah satu container - traefik akan memanggil resolver secara otomatis. contoh kita pakai letsencrypt, LE akan fetch certificate utk kita.

```
version: '3.3'
services: 
  traefik:
    image: traefik:v2.6
    command:
      #- "--log.level=DEBUG"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443" #1
      - "--certificateresolvers.myresolver.acme.tlschallenge=true" #2
      - "--certiificateresolvers.myresolver.acme.email=your@email.com" #3
      - "--certificateresolvers.myresolver.acme.storage=/letsencrypt/acme.json" #4
    ports:
      - "80:80"
      - "8080:8080"
      - "443:443" #5
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./letsencrypt:/letsencrypt" #6
  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`domainku.com`)"
      - "traefik.http.routers.whoami.entrypoints=web,websecure" #7
      - "traefik.http.routers.whoami.tls.certresolver=myresolver" #8
```

keterangan:
1. kita tambah SSL entrypoint, listen HTTPS port 443
2. enable acme challenge utk `myresolver` sbg certificate resolver name
3. tambahkan email utk LE
4. path ke file json utk menyimpan certificate
5. kita expose port 443
6. kita mount volume certificate di container ke file di localdisk, jadi update traefik tidak perlu mendownload ulang semua cert
7. tambahkan entrypoint `websecure` ke service sehingga kita bisa akses via HTTP dan HTTPS
8. kita bertahu traefik utk menggunakan `myresolver` utk mendapat SSL certificate utk service ini.

run `docker-compose up -d` utk me-fetch otomatis certificate & menggunakannya.

kita bisa cek traefik dashboard utk melihat status SSL di router:

![img](https://accesto.com/blog/static/302ad3456a13f6d804a738c97627a828/69476/traefik-ssl-dashboard-view.png)

sekarang kita punya reverse proxy di docker dg SSL termination.

## fitur Middleware

jika kita ingin membuat secure access utk endpoint tertentu. kita bisa gunakan Whitelist ip yg datang atau menggunakan username & password. kita bisa buat di level app (tiap app), tapi disini kita buat di level reverse proxy.

### IP Whitelist

Web app security is key, jadi bagaimana menambahkan ip whitelist? kita tambah 2 labels baru:

```
      - "traefik.http.middlewares.whoami-filter-ip.ipwhitelist.sourcerange=192.168.1.1/24,127.0.0.1/32"
      - "traefik.http.routers.whoami.middlewares=whoami-filter-ip"
```

kita buat middleware dg nama `whoami-filter-ip` (harus uniq). ip yg kita whitelist akan di ijinkan utk akses ke service, sumber ip lain akan mendapat `403 Forbidden`.

### Basic Auth

menambahkan basic auth utk mengamankan web/service.

```
      - "traefik.http.middlewares.whoami-auth.basicauth.users=test:$$apr1$$ra8uoeq5$$HqiATqC5edVVEXznsNiVV/,test2:$$apr1$$8ol2akty$$BW.Fsa.K3tc1DzcJ6l9ql1"
      - "traefik.http.routers.whoami.middlewares=whoami-auth"
```

kita generate user & password di console dg perintah:

```
echo $(htpasswd -nb user password) | sed -e s/\\$/\\$\\$/g
```

![img](https://accesto.com/blog/static/8c2e120c4db68e412ce0f9e544bfbd21/0940f/basic-auth-example.png)

`sed` me-replace single `$` dengan dual `$$` agar docker-compose tidak menganggapnya sbg env variable.

> traefik tidak hanya support HTTP, juga TCP. misal utk database. juga dg built-in middlewares yg berbeda.
> kita bisa tambah custom headers, rate limiting, redirects, retries, compression, cirtuit breaker, custom error pages etc.











