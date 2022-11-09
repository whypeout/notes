# Install Docker

```sh
sudo -i
apt update -y && apt upgrade -y
apt install -y docker.io  # mostly not latest version release by docker
docker version
```

# Docker Basic Commands

```sh
docker pull debian        # pull "latest" image default without tags
docker pull ubuntu:20.04  # pull ubuntu tags version 20.04

# run create container
uname -a
docker run debian uname -a

# interactive shell inside container
docker run -it --name=debian debian /bin/bash
docker exec -it debian /bin/bash
# detach interactive shell with: Ctrl+P then Ctrl+Q

# show running & all containers
docker ps       # only show running container
docker ps -a    # show all containers, including stopped
docker ps -aq   # show all containers id only

# connect to container's session
docker attach debian

# kill container
docker kill debian
```

# Docker Images

```sh
docker images
docker image ls

docker run debian /bin/bash -c "apt-get update; apt-get install -y nginx"
docker ps -a | head -2

docker commit <container-id> myserver.lan/debian-nginx:latest
docker images
docker run myserver.lan/debian-nginx /usr/bin/which nginx
```

# Access Container Services

```sh
docker run -t -d -p 8081:80 myserver.lan/debian-nginx /usr/bin/nginx -g "daemon off;"
docker ps
docker exec <container-id> /bin/bash -c 'echo "Nginx on Docker Container' > /var/www/html/index2.html
curl http://localhost:8081/index2.html
```

# Dockerfile

```sh
vi Dockerfile
# create new image from other image
FROM debian
MAINTAINER ServerWorld <admin@myserver.lan>

RUN apt-get update -y
RUN apt-get -y install tzdata \
    && apt-get -y install apache2
RUN echo "Dockerfile Test on Apache2" > /var/www/html/index.html
EXPOSE 80
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]

# build image => docker build -t [image-name]:[tag] .
docker build -t myserver.lan/debian-apache2:latest .
docker images

docker run -d -p 8082:80 myserver.lan/debian-apache2
docker ps

curl localhost:8082
```

Instruction:
| Keywords | Keterangan |
|-----------|-----------|
| FROM | base image utk image baru |
| MAINTAINER | author field utk image baru |
| RUN | command utk dijalankan ketika build image baru |
| CMD | command utk dijalankan ketika container dibuat |
| ENTRYPOINT | command utk dijalankan ketika container dibuat |
| LABEL | tambahkan metadata kedalam image |
| EXPOSE | port yg digunakan image ketika container dibuat |
| ENV | tambahkan env var |
| ADD | copy new files, folder atau remote URL |
| COPY | copy new files/folder. tidak bisa remote URL & tidak extract archive |
| VOLUME | mount point dg nama spesifik dan tandai mounted volumes dari native host atau container lain |
| USER | set Username atau UID |
| WORKDIR | working directory |

# External Storage

ketika container di remove, data didalamnya akan hilang, maka diperlukan external filesystem
didalam container sbg persistent storage jika diperlukan.

- Mount directory Docker Host kedalam container

```sh
mkdir -p /var/lib/docker/disk01
echo "persistent storage" >> /var/lib/docker/disk01/testfile.txt
docker run -it -v /var/lib/docker/disk01:/mnt debian /bin/bash
df -hT
cat /mnt/testfile.txt
```

- Menggunakan Docker Data Volume

```sh
docker volume create debain_volume1
docker volume ls
docker volume inspect volume1
docker run -it -v volume1:/mnt debian /bin/bash
df -hT
echo "Docker Volume persistent" > /mnt/testfile.txt
exit
docker rm debian-old

docker run -v volume1:/var/volume1 debian cat /var/volume1/testfile.txt

docker volume rm volume1
```

> volume tidak bisa di-remove jika masih ada container yg menggunakan.
> hapus container dulu, baru hapus volume nya

# NFS Network FileSystem

menggunakan NFS Server sbg external storage utk container. misal /home/nfsshare pada server-nfs.lan

> Setup NFS Server terlebih dulu, misal Synology NAS, OpenMediaVault (Debian),
> TrueNAS Scale/Core, UnRaid

```sh
docker volume create \
--opt type=nfs \
--opt o=addr=10.0.0.35,rw,nfsvers=4 \
--opt device=:/home/nfsshare nfs-volume

docker volume ls
docker volume inspect nfs-volume

docker run -it -v nfs-volume:/nfsshare debian /bin/bash
df -hT /nfsshare
echo "NFS Volume Test" > /nfsshare/testfile.txt
cat /nfsshare/testfile.txt
```

# Image Registry

Docker-Registry for Private Registry to Store Docker Images Locally

Install Registry

```sh
apt install -y docker-registry
```

Configure Registry

```
vi /etc/docker/registry/config.yml
```

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/docker-registry
  delete:
    enabled: true
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
#auth:
# htpasswd:
#   htpasswd:
#     realm: basic-realm
#     path: /etc/docker/registry
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

Restart Registry Service

```sh
systemctl restart docker-registry
```

Percobaan Akses dari client, untuk HTTP Connection biasanya perlu tambahan setting [insecure-registries](#)

```json
# vi /etc/docker/daemon.json
{
  "insecure-registries":
    [
      "docker.internal:5000",
      "myserver.lan:5000"
    ]
}
```

Coba Push, Pull pada registry lokal

```sh
systemctl restart docker

# Push
docker tag debian myserver.lan:5000/debian:lokal
docker push myserver.lan:5000/debian:lokal
docker images

# Pull dari docker host lain
docker pull myserver.lan:5000/debian:lokal
docker images
```

# Enable Basic Auth

```sh
apt install -y apache2-utils
vi /etc/docker/registry/config.yml
# [auth] section
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/.htpasswd
# save & close
systemctl restart docker-registry
# gunakan -c pada pembuatan pertama file passwd
htpasswd -cB /etc/docker/registry/.htpasswd debian

# coba pull tanpa auth
docker pull myserver.lan:5000/debian:lokal

docker login myserver.lan:5000
Username:
Password:
credentials stored in /root/.docker/config.json
Login Succeeded

docker pull myserver.lan:5000/debian:lokal
docker images
```

Using TLS/SSL to secure HTTP Registry

```sh
mkdir /etc/docker/certs.d
cp -p /etc/letsencrypt/live/myserver.lan/{fullchain,privkey}.pem /etc/docker/certs.d/
chown docker-registry /etc/docker/certs.d/{fullchain,privkey}.pem
vi /etc/docker/registry/config.yml

# add [tls] section under [http] section
http:
  addr: :5000
  tls:
    certificate: /etc/docker/certs.d/fullchain.pem
    key: /etc/docker/certs.d/privkey.pem
...
systemctl restart docker-registry

# kita bisa hapus [insecure-registries] setelah menggunakan ssl/tls
docker pull myserver.lan:5000/debian:lokal
docker images
```

# Docker Network

```sh
docker network ls
docker network inspect bridge
docker network host
docker network none

docker run debian /bin/bash -c "apt-get update; apt-get -y install iproute2; ip route"

# create network [net01] with [192.168.100.0/24] subnet
docker network create --subnet 192.168.100.0/24 net01
docker network ls

# run container dg spesifik network
docker run --net net01 debian /bin/bash -c "apt-get update; apt-get -y install iproute2; ip route"
docker ps
docker exec <debian-id> /bin/bash -c "apt-get update; apt-get -y install iproute2; ip route"

# attach container dg ip addr spesifik
docker network connect --ip 192.168.100.10 net01 <debian-id>
docker exec <debian-id> ip route

# disconnect network
docker network disconnect net01 <debian-id>
docker exec <debian-id> ip route

# remove docker network, pastikan disconnect semua container dulu
docker network rm net01

# remove semua network yg tidak dipakai
docker network prune

# connect container ke network host, bukan bridge
docker run -d --net host myserver.lan:5000/debian:lokal
docker ps
ss -napt
curl localhost
```

# Docker Compose

```sh
apt -y install docker-compose
vi Dockerfile
# web service container
FROM debian
MAINTAINER myserver <admin@myserver.lan>

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get -y install apache2

EXPOSE 80
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]

# define app config
vi docker-compose.yml
version: '3'
services:
  db:
    image: mariadb
    volumes:
      - /var/lib/docker/disk01:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: mypwdOK
      MYSQL_USER: newUser
      MYSQL_PASSWORD: newPwdOK
      MYSQL_DATABASE: wordpress
    ports:
      - "3306:3306"
  web:
    build: .
    ports:
      - "80:80"
    volumes:
      - /var/lib/docker/disk02:/var/www/html

# build and run
docker-compose up -d
docker ps

# akses mysql server container
mysql -h 127.0.0.1 -u root -pmypwdOK -e "show variables like 'hostname';"
mysql -h 127.0.0.1 -u newUser -pnewPwdOK -e "show databases;"
echo "Hello Docker Compose World"" > /var/lib/docker/disk02/index.html
curl 127.0.0.1
```

Basic Docker Compose Lain

```sh
# jalankan dari folder tempat docker-compose.yml berada
docker-compose ps
docker-compose logs
docker-compose logs -f web
docker-compose exec db /bin/bash
docker-compose stop
docker-compose up -d web
docker-compose ps
docker-compose rm

# remove semua container sekaligus
docker-compose down
```

# Docker Swarm Cluster

buat Docker Swarm Cluster dg 3 nodes. ada 2 roles dalam Swarm Cluster:
- Manager Nodes (manager 10.0.0.51)
- Worker Nodes  (worker 1 10.0.0.52, worker 2 10.0.0.53)

Disable Live-Restore pada semua Node. Swarm Cluster tidak support live-restore

```sh
vi /etc/docker/daemon.json
{
  "live-restore": false
}
systemctl restart docker
```

Swarm Init

```sh
docker swarm init
## to add a worker to this swarm, run the following command:
docker swarm join --token <join-token> 10.0.0.51:2377
## to add manager, run:
docker swarm <join-token> manager

## connect all worker to manager
node1:/# docker node ls

node1:/# vi Dockerfile
FROM debian
MAINTAINER manager <admin@myserver.lan>
RUN apt-get update
RUN apt-get -y install nginx
RUN echo "Nginx on node01" > /var/www/html/index.html
EXPOSE 80
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
node1:/# docker build -t nginx-server:latest .
node1:/# docker images

# create service with 2 replicas
node1:/# docker service create --name=swarm_cluster --replicas=2 -p 80:80 nginx-server:latest
node1:/# docker service ls
node1:/# docker service inspect swarm_cluster --pretty
node1:/# docker service ps swarm_cluster
node1:/# node1.myserver.lan #try many times to see the response diff
node1:/# docker service scale swarm_cluster=3
node1:/# docker service ps swarm_cluster
```

sources:
- [https://www.server-world.info/en/note?os=Debian_11&p=docker&f=2](https://www.server-world.info/en/note?os=Debian_11&p=docker&f=2)
