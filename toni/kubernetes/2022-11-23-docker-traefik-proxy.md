# Traefik v2 sbg Reverse Proxy utk Docker Containers di Ubuntu 20.04

sources:
- [https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04)

## Pre Reqs

- ubuntu 20.04 server
- install docker di ubuntu 20.04 server
- install docker compose di ubuntu 20.04
- DNS A records utk db-admin.domain blog.domain dan monitor.domain ke IP Public ubuntu server

## 1 Traefik Config & Running

kita akan pakai htpasswd utk membuat basic auth (install apache2-utils) login ke dashboard monitoring & heatlh check.

```
sudo apt-get install apache2-utils
htpasswd -nb admin secure_password
# output: admin:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/
```

kita perlu dua file: `traefik.toml` dan `traefik_dynamic.toml` kita pakai format TOML (mirip INI dg standar).
traefik punya beberapa `providers` : `api`, `docker`, dan `acme` (support TLS cert dg Lets Encrypt)

```
# nano traefik.toml
[entryPoints]
  [entryPoints.web]
    address = ":80"
    [entryPoints.web.http.redirections.entryPoint]
      to = "websecure"
      scheme = "https"
  [entryPoints.websecure]
    address = ":443"

[api]
  dashboard = true
  
[certificatesResolvers.lets-encrypt.acme]
  email = "your_email@domain"
  storage = "acme.json"
  [certificatesResolvers.lets-encrypt.acme.tlsChallenge]
  
[providers.docker]
  watch = true
  network = "web"

[providers.file]
  filename = "traefik_dynamic.toml"
```

disini kita redirect semua trafik HTTP port 80 agar dihandle lewat TLS HTTPS port 443.
kita juga enable akses `api` dan dashboard.
terakhir kita enable letsencrypt utk menghandle SSL/TLS Cert yg valid dg membuat certificate resolver bertipe `acme`.
acme adl protokol utk komunikasi dg letsencrypt utk memanage cert. LE perlu registrasi dg valid email addr (`email` key) dan kita simpan info dari LE dlm file `acme.json`, `acme.tlsChallenge` meminta LE agar melakukan Challenge pada port 443.
jangan lupa enable providers utk docker. traefik akan bertindak sbg proxy utk setiap docker container, dg memantau `watch` container pada network `web` yg akan kita buat.
karena traefik tidak bisa menggunakan static config & dinamik konfig bersamaan, kita gunakan konfig dinamis kita dg `file` providers didalam static konfig kita.

konfig ini akan berisi `middlewares` dan `routers`. untuk men-secure dashboard dg password kita perlu ubah router API dan menambah middleware utk menghandle HTTP basic auth.
middleware dikonfig per-protocol, disini kita pakai protocol HTTP, jadi kita pakai `http.middlewares` dg tipe `basicAuth` yg kita namai `simpleAuth`.

```
# nano traefik_dynamic.toml
# 1
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/"
  ]

# 2
[http.routers.api]
  rule = "Host(`monitor.domain`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

2. kita pakai `http.routers` utk konfig router `api` providers. kita konfig domain utk dashboard dg `key` rule berupa host match, menggunakan entrypoints `websecure` dan tambahkan middleware `simpleAuth`
   `web` entrypoints handle port `80`, sedangkan `websecure` entrypoint handle port 443 utk TLS/SSL. kita otomatis redirect semua trafik port `80` ke port `443` utk memaksa semua koneksi secure.
   
terakhir konfig service, enable tls dan `certResolver` ke `lets-encrypt`. service adl final step utk memutuskan dimana request terakhir dihandle. service `api@internal` adl built-in service dibelakang API yg kita expose. seperti routers dan middlewares, service bisa dikonfig didalam file ini, tapi hasilnya tidak akan seperti yg kita inginkan.

## 2 Running Traefik Container

kita buat docker network yg akan digunakan containers bersama traefik proxy. 

```
docker network create web
```

setelah kita buat traefik container, kita tambahkan ke network ini. selanjutnya kita tambah containers/app lain ke network ini agar dapat menggunakan proxy.

kita buat file kosong utk menyimpan data LetsEncrypt utk traefik:

```
touch acme.json
chmod 600 acme.json
```

kita run container traefik:

```
docker run -d \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $PWD/traefik.toml:/traefik.toml \
-v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
-v $PWD/acme.json:/acme.json \
-p 80:80 \
-p 443:443 \
--network web \
--name traefik \
traefik:v2.2
```

didalam container, command `traefik` akan pertama kali dijalankan. kita dapat menambahkan args ke command ketika menjalankan container, tapi sudah kita konfig semua di file `traefik.toml`.

kita bisa akses dashboard utk melihat health dari containers. juga utk melihat secara visual routers, services dan middlewares yg terdaftar di traefik. coba akses dg browser ke `https://monitor.domain/dashboard/` (`/` diakhir diperlukan).

kita akan diminta memasukan username (admin) dan password yg kita konfig di Step 1.

![img](https://assets.digitalocean.com/articles/67541/traefik_2_empty_dashboard.1.png)

kita dapat lihat hasil konfig kita berupa middlewares, routers & services.
kita telah sukses menjalankan traefik proxy, terkonfigurasi utk bekerja dg docker dan monitor containers lain. selanjutnya kita tambah container lain ke proxy.

## 3 Menambahkan Containers ke Traefik

kita akan jalankan 2 services:
- wordpress blog container
- adminer db management server

```
# nano docker-compose.yml
version: "3"

networks:
  web:
    external: true
  internal:
    external: false
   
services:
  blog:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
      - traefik.http.routers.blog.rule=Host(`blog.domain`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal
      - web
    depends_on:
      - mysql
```

karena network `web` kita buat secara manual, kita buat `external: true`. lalu kita buat network `internal` baru utk database, karena kita tidak expose database dg proxy.

kita buat container/service `blog` dg image official dari wordpress. kita kosongkan env `WORDPRESS_DB_PASSWORD` utk menghindari menulis password didalam docker-compose.yml file, tapi kita pakai shell env saat kita jalankan docker-compose.

config values pada `labels` akan digunakan oleh traefik utk mengkonfig containers. docker tidak menggunakan labels sama sekali.

- `traefik.http.routers.blog.rule=Host("blog.domain")` buat router baru (bernama `blog`), nantinya req match host ini akan di route ke container ini
- `traefik.routers.blog.tls=true` enable TLS utk router ini
- `traefik.routers.blog.tls.certResolver=lets-encrypt` kita pakai `lets-encrypt` sbg resolver utk mendapatkan cert utk router ini. sebelumnya kita sudah buat konfig resolver ini di traefik.toml & traefik_dynamic.toml
- `traefik.port` port yg digunakan container, dimana trafik akan merutekan trafik ke container

dg konfig ini, semua trafik yg dikirim ke docker host port 80 dan 443 dg domain `blog.domain` akan di rutekan ke container `blog`.  
container `blog` digabungkan ke 2 network berbeda, shg traefik bisa menemukannya via network `web` dan berkomunikasi dg container database lewat `internal` network.  
terakhir, key `depends_on` membuat docker-compose menjalankan `blog` **setelah** `mysql` berjalan.

kita tambahkan container mysql ke docker compose:

```
# docker-compose.yml##mysql container
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD:
    networks:
      - internal
    labels:
      - traefik.enable=false
```

tambahkan env `MYSQL_ROOT_PASSWORD` pada shell saat menjalankan command docker-compose.
kita tidak ingin meng-expose `mysql` lewat traefik, jadi `mysql` hanya akan terhubung di network `internal` dg `blog`.
karena traefik memonitor docker via docker.sock, secara default traefik akan meng-expose semua container, kita perlu disable expose utk `mysql` dg `traefik.enable=false`

terakhir kita tambah container `adminer`:

```
  adminer:
    image: adminer:4.6.3-standalone
    labels:
      - traefik.http.routers.adminer.rule=Host(`db-admin.domain`)
      - traefik.http.routers.adminer.tls=true
      - traefik.http.routers.adminer.tls.certresolver=lets-encrypt
      - traefik.port=8080
    networks:
      - internal
      - web
    depends_on:
      - mysql
```
kita pakai official image adminer. konfig `network` dan `depends_on` sama dg container `blog`.
`traefik.http.routers.adminer.rule=Host("db-admin.domain")`  container ini dpt diakses menggunakan domain/host `db-admin.domain`, traefik akan me-rutekan req ke port `8080` (port default container adminer)

final docker-compose.yaml

```
version: "3"

networks:
  web:
    external: true
  internal:
    external: false

services:
  blog:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
      - traefik.http.routers.blog.rule=Host(`blog.domain`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal
      - web
    depends_on:
      - mysql

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD:
    networks:
      - internal
    labels:
      - traefik.enable=false

  adminer:
    image: adminer:4.6.3-standalone
    labels:
    labels:
      - traefik.http.routers.adminer.rule=Host(`db-admin.domain`)
      - traefik.http.routers.adminer.tls=true
      - traefik.http.routers.adminer.tls.certresolver=lets-encrypt
      - traefik.port=8080
    networks:
      - internal
      - web
    depends_on:
      - mysql
```

kita jalankan semua container dg env tentunya:

```
export WORDPRESS_DB_PASSWORD=secure_database_password
export MYSQL_ROOT_PASSWORD=secure_database_password
docker-compose up -d
```

buka traefik admin dashboard utk memonitor services & routers

![img](https://assets.digitalocean.com/articles/67541/traefik_2_populated_dashboard.1.png)
![img](https://assets.digitalocean.com/articles/67541/traefik_2_http_routers.1.png)

buka `blog.domain`, kita akan diarahkan utk menggunakan koneksi TLS dan menyelesaikan setup wordpress.

![img](https://assets.digitalocean.com/articles/67541/traefik_2_wordpress_setup.1.png)

buka `db-admin.domain` utk mengakses adminer. container `mysql` tidak di expose ke luar, tapi container `adminer` punya akses lewat `internal` docker network, bisa menggunakan hostname `mysql` utk terkoneksi.

dihalaman login adminer, masukkan **username** `root` dan **server** `mysql`, utk **password** gunakan value dari env `MYSQL_ROOT_PASSWORD`. **database** bisa kita kosongkan. kita tekan **Login**.

![img](https://assets.digitalocean.com/articles/67541/traefik_2_adminer_screen.1.png)

utk memonitor app kita lewat dashboard buka `monitor.domain`.





