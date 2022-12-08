# KinD :: Mengakses Kubernetes Service dari Host

sources:
- [https://dustinspecker.com/posts/resolving-kubernetes-services-from-host-when-using-kind/](https://dustinspecker.com/posts/resolving-kubernetes-services-from-host-when-using-kind/)

TOC

1. verifikasi konfigurasi DNS pada host
2. membuat cluster kubernetes dg KinD
3. deploy service hello-world
4. add route langsung trafik ke pods didalam cluster
5. add route langusng trafik ke service didalam cluster
6. cleanup host env
7. menggunakan docker, tanpa mengubah host

KinD adl salah satu tools utk dev local & testing kubernetes. ada banyak keuntungan kind seperti setiap node adl docker container sehingga mudah melakukan setup dan membedah cluster.
negatifnya, menjalankan seluruh cluster didalam docker container membuat issue/masalah yg tidak biasanya terjadi, misal pada `minikube`.

salah satu masalah ketika terlalu banyak menjalankan service, kita tidak bisa me-resolve dari host. sayangnya, kita bisa buat DNS konfig & routing utk menyelesaikan issue ini.

> sumber post menggunakan kind 0.9.0 dan kubectl v1.19.4 berjalan pada ubuntu 19.10. tidak dites pada macos dan windodws.

> post lanjutan utk me-resolve kubernetes service lewat docker pada kind cluster tanpa modifikasi host. tidak terbatas pada linux.

## 1. konfigurasi dns pada host

```
systemd-resolve --status | grep 'DNS Servers' --after 5
DNS Servers: 10.96.0.10
             192.168.0.1
DNS Domain: svc.cluster.local
            cluster.local
```            

pastikan ip dns `10.96.0.10` ada di list pertama/paling atas. jika tidak, kita bisa konfig, misal lewat netplan:
```
# nano /etc/netplan/netplan.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp4s0:
      dhcp4: true
      nameservers:
        search: ["svr.cluster.local", "cluster.local"]
        addresses: ["10.96.0.10","192.168.0.1"]
```
search nameserver tidak penting, tapi akan memudahkan me-resolve `service_name.namespace` req. `enp4s0` adl nama interface yg bisa kita cek lewat:
```
ip addr
```
setelah dns server `10.96.0.10` return oleh `systemd-resolves` output. kita bisa lanjut.

> ip `10.96.0.10` adl ip kube-dns service yg akan kita buat di cluster kita. harus di list pertama, atau req ke `service_name.namespace.svr.cluster.local` akan gagal.

## 2. Membuat cluster kubernetes dg kind

```
kind create cluster
```

setelah selesai, kita akan mempunyai cluster berisi satu node didalam docker container.

## 3. Deploy service Hello-world

kita akan menggunakan nginx demo sbg service yg melayani http req. utk me-return plain text nanti.
buat pod dan service di cluster:
```
kubectl run hello --expose --image nginxdemos/hello:plain-text --port 80
```

## 4. Add Route to Pods

ambil IP pods hello
```
kubectl get pods --namespace default --output wide
```
perhatikan kolom ip address pod. misal `10.244.0.5`.
coba akses pod lewat docker:
```
docker exec kind-control-plane curl 10.244.0.5

# kurleb outpunya begini:
Server address: 10.244.0.5:80
Server name: hello-5ccfd6b56f-ch7hv
Date: 09/May/2020:21:36:58 +0000
URI: /
Request ID: b9004e01688d3d2659c2169851a85a9c
```
kita tidak bisa mengakses pods langsung dari host dg. Ctrl+C utk meng-cancel curl
```
curl 10.244.0.5
```

kita perlu tahu ip addr docker container:
```
docker container inspect kind-control-plane --format '{{ .NetworkSettings.Network.kind.IPAddress }}'
```
misal ip cluster/container: `172.18.0.2`.
utk dpt mengakses pods didalam container, kita perlu tambahkan route di host:
```
# melihat route saat ini
ip route

# menambah route ke pods 
sudo ip route add 10.244.0.5 via 172.18.0.2

# cek kembali route setelah ditambahkan
ip route

# coba curl kembali ke pods ip
curl 10.244.0.5

# ambil range ip pods
kubectl get node kind-control-plane --output jsonpath='{@.spec.podCIDR}'

# selain per-ip, kita bisa tambahkan range ip pods
sudo ip route delete 10.244.0.5
sudo ip route add 10.244.0.0/24 via 172.18.0.2
```
pastikan utk menggunakan ip cluster dan pods yg benar.
jika cluster kita ada beberapa nodes, kita bisa tambah route utk setiap nodes/docker container utk mengakses pod CIDR didalamnya

## Add Route to Service

```
# ambil ip service, misal service ip 10.109.139.197 yg merupakan CIDR 10.96.0.0/12
kubectl get service hello --namespace default

# tambahkan route
sudo ip route add 10.96.0.0/12 via 172.18.0.2
curl 10.109.139.197

# kita bisa juga coba dg domain/dns
curl hello.default.svc.cluster.local
```

## Cleanup

hapus semua route jika sudah tidak diperlukan.misal kita buat ulang cluster
```
sudo ip route delete 10.96.0.0/12
sudo ip route delete 10.244.0.0/24
kind delete cluster
```

## Menggunakan docker tanpa merubah host (ip route)
