# Mengakses Kubernetes service via Docker

sources:
- [https://dustinspecker.com/posts/using-docker-to-resolve-kubernetes-services-in-a-kind-cluster/](https://dustinspecker.com/posts/using-docker-to-resolve-kubernetes-services-in-a-kind-cluster/)

TOC

1. membuat cluster kind & deploy hello service
2. menjalankan docker container
3. mengubah route pada docker container
4. membuat request ke kubernetes service didalam kind cluster
5. kesimpulan

pada post sebelumnya [https://dustinspecker.com/posts/resolving-kubernetes-services-from-host-when-using-kind/](https://dustinspecker.com/posts/resolving-kubernetes-services-from-host-when-using-kind/),
kita modifikasi host dns dan route (`/etc/resolv.conf`).

diakhir nanti kita akan membuat docker container bisa mengakses service request ke `http://hello.default.svc.cluster.local`.

## Membuat cluster dan deploy hello service

```
kind create cluster --wait 300s

kubectl run hello --expose --image nginxdemos/hello:plain-text --port 80
```

## Menjalankan docker container

```
docker run \
  --cap-add NET_ADMIN \
  --detach \
  --dns 10.96.0.10 \
  --dns-search svc.cluster.local \
  --dns-search cluster.local \
  --interactive \
  --name docker-kind-demo \
  --net kind \
  --rm \
  --tty \
  curlimages/curl:7.71.0 cat
```

- net_admin : mengijinkan merubah ip addr & route
- dns : server gunakan milik kind-cluster-plane
- dns-search : agar otomatis resolve domain pada kind-cluster-plane
- net kind : gunakan network yg sama dg kind-cluster-plane
- cat : jalankan cmd ini, agar container tidak langsung "stopped"

## Mengubah route pada docker container

kita buat docker container "docker-kind-demo" utk langsung mengarahkan trafik ke kind-cluster. dapatkan ip kind-cluster:
```
docker container inspect kind-control-plane \
  --format '{{ .NetworkSettings.Network.kind.IPAddress }}'
```
misal output: 172.18.0.2
```
docker exec \
  --interactive \
  --tty \
  --user 0 \
  docker-kind-demo ip route add 10.96.0.0/12 via 172.18.0.2
```
kita arahkan reqest utk service kubernetes cluster (`10.96.0.0/12`) lewat `172.18.0.2`.
> perintah `ip route add` harus dijalankan sbg root `--user 0`.

## Request ke kubernetes service didalam cluster kind

```
docker exec \
  --interactive \
  --tty \
  docker-kind-demo curl http://hello.default.svc.cluster.local
```

sekarang kita punya docker container yg bisa berkomunikasi dg kubernetes service didalam cluster tanpa mengganggu host.

## Kesimpulan

kita telah membuat cluster kubernetes via kind menjalankan hello service.
kita kemudian menjalankan docker container terkonfigurasi utk berkomunikasi dg service berjalan didalam cluster kind.





