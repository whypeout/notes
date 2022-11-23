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
  email = "your_email@your_domain"
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
  rule = "Host(`monitor.your_domain`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

2. kita pakai `http.routers` utk konfig router `api` providers. kita konfig domain utk dashboard dg `key` rule berupa host match, menggunakan entrypoints `websecure` dan tambahkan middleware `simpleAuth`
   `web` entrypoints handle port `80`, sedangkan `websecure` entrypoint handle port 443 utk TLS/SSL. kita otomatis redirect semua trafik port `80` ke port `443` utk memaksa semua koneksi secure.
   
terakhir konfig service, enable tls dan `certResolver` ke `lets-encrypt`. service adl final step utk memutuskan dimana request terakhir dihandle.




