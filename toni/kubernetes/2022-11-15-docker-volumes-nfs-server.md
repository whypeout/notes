# Docker Volumes :: Peristent Storage using NFS Server

souces:
- [https://phoenixnap.com/kb/nfs-docker-volumes](https://phoenixnap.com/kb/nfs-docker-volumes)

umumnya docker container menyimpan data didalam container, jika kita remove container, maka data didalamnya pun ikut terhapus.
docker volume merupakan mekanisme utk penyimpanan tetap/persistent storage utk docker container.
kita bisa mengatur volumes agar menggunakan directory yg ada di filesystem host utk di (bind) mount kedalam container.
sehingga data bisa diakses baik dari host sistem dan container.

docker juga mengijinkan user me-mount directory share dari NFS server. volume ini menggunakan docker NFS driver (bukan host driver),
sehingga tidak perlu me-mount NFS directory di host sistem.

## Create Docker NFS Volume

```
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=[ip-address],rw \
  --opt device=:[path-to-directory] \
  [volume-name]
```
  
keterangan:
- volume type nfs
- use write mode (rw), not read-only mode (ro)
- ip-address dari nfs server
- path ke directory yg di share di server

contoh:

```
docker volume create --driver local \
--opt type=nfs \
--opt o=addr=192.168.1.8,rw \
--opt device=:/mnt/nfsdir \
nfs-volume

docker volume ls

docker volume inspect nfs-volume
```

![image](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-docker-volume-inspect-nfs-volume.png)

## Mount NFS di dalam Container

kita perlu install paket nfs-common di host system.

```
apt update
apt install nfs-common
```

![common](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-sudo-apt-install-nfs-common-for-docker-nfs-volume.png)

```
docker run -d -it \
  --name [container-name] \
  --mount source=[volume-name],target=[mount-point]\
  [image-name]
  
docker run -d -it --name debian-test --mount source=nfs-volume,target=/mnt debian

docker inspect debian-test

docker exec -it debian-test /bin/bash
/# ls -la /mnt
touch /mnt/docker1.txt
```

![nfs](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-docker-run-d-it-name-debian-test-etc.png)

![inspect](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-docker-inspect-nfs-volume.png)

![ls/mnt](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-ls-mnt-on-mounted-docker-nfs-volume.png)

![touch](https://phoenixnap.com/kb/wp-content/uploads/2021/12/output-from-ls-for-docker-nfs-volume-mount.png)

## Docker Compose :: Mount NFS Volume

```yaml
# nano docker-compose.yml
version: "3.2"
services:
  [service-name]:
    image: [docker-image]
    ports:
      - "[port]:[port]"
    volumes:
      - type: volume
        source: [volume-name]
        target: /nfs
        volume:
          nocopy: true
volumes:
  [volume-name]:
    driver_opts:
      type: "nfs"
      o: "addr=[ip-address],nolock,soft,rw"
      device: ":[path-to-directory]"
```

> `nolock` dan `soft` memastikan docker tidak freeze ketika koneksi ke NFS server hilang





