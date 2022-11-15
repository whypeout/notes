# Sharing Directory via SMB on Docker Container

sources:
- [http://www.freekb.net/Article?id=3428](http://www.freekb.net/Article?id=3428)
- [https://github.com/dperson/samba](https://github.com/dperson/samba)
- [https://shfahimi.net/2020/07/04/setting-up-your-home-media-server-2/](https://shfahimi.net/2020/07/04/setting-up-your-home-media-server-2/)


```
docker pull dperson/samba

# nano Dockerfile
FROM samba:latest

docker build . --tag samba:latest

docker images

docker run -d -p 139:139 -p 445:445 \
-v ./samba:/etc/samba \
-v ./samba/share:/usr/local/share \
--restart=unless-stopped --name=samba \
dperson/samba

# nano ./samba/smb.conf utk volume dg tdbsam (trivial database sam) backend
[global]
workgroup = SAMBA
security = user
passdb backend = tdbsam
printing = bsd
printcap name = /dev/null

[homes]
comment = Home Directories
valid users = %S, %D%w%S
browseable = No
read only = No
inherit acls = Yes

[share]
path = /usr/local/share
browsable = yes
public = yes
writeable = yes

# utk volume shared pada Docker system dg SELinux type etc_t
~]$ ls -lZ /usr/local/docker/samba/
drwxrwxrwx. 4 root root system_u:object_r:usr_t:s0 4096 Oct 25 02:37 share

# pastikan container sudah running
~]# docker container ls -a
CONTAINER ID   IMAGE           COMMAND                 CREATED      STATUS      PORTS                            
ba2fff144f7f   dperlson/samba  "/sbin/tini -- /usr/â€¦"  3 hours ago  Up 3 hours  0.0.0.0:139->139/tcp, :::139->139/tcp, 137-138/udp, 0.0.0.0:445->445/tcp, :::445->445/tcp  samba

# logs container sudah ready
~]# docker logs samba
smbd version 4.12.2 started.
Copyright Andrew Tridgell and the Samba Team 1992-2020
daemon_ready: daemon 'smbd' finished starting up and ready to serve connections

# buat user baru
docker exec -it samba /bin/bash
useradd -p itsasecret -d /home/john.doe -s /bin/bash john.doe

# cek user yg sudah dibuat
~]# docker exec samba cat /etc/passwd
john.doe:x:1000:1000::/home/john.doe:/bin/bash

# daftarkan user kedalam database samba
~]# docker exec -it samba bash
bash-5.0# smbpasswd -a john.doe
New SMB password:
Retype new SMB password:
Added user john.doe.

# verify user sudah terdaftar di database smb
~]# docker exec samba pdbedit --list --smbpasswd-style
john.doe:1000:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:43A29EF878C2498C6960672E21CA9B9D:[U          ]:LCT-616C00EF:

~]# docker exec samba cat /etc/samba/smb.passwd
john.doe:1000:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:43A29EF878C2498C6960672E21CA9B9D:[U          ]:LCT-616C00EF:
 
# coba koneksi ke server
smbclient --list //$(hostname -s)/share --user john.doe
```

README.md

```sh
# default cmd
docker run -it -p 139:139 -p 445:445 -d dperson/samba -p

# set local storage
docker run -it -d --name samba -p 139:139 -p 445:445 \
-v /mnt/disk-1:/mnt \
dperson/samba -p

# help
docker run --rm -it dperson/samba -h

# example run
docker run -it -e TZ=Asia/Jakarta -p 139:139 -p 445:445 -d \
-v /mnt/disk-1:/mnt -m 1024m \
dperson/samba -p \
-u "user1;badpass1" \
-u "user2;badpass2" \
-s "public;/mnt/share" \
-s "users;/mnt/users;no;no;no;user1,user2" \
-s "user1 private share;/mnt/user1;no;no;no;user1" \
-s "user2 private share;/mnt/user2;no;no;no;user2"
```

- [https://github.com/dperson/samba/blob/master/docker-compose.yml](https://github.com/dperson/samba/blob/master/docker-compose.yml)

```yaml
# nano docker-compose.yml
version: '3.4'

services:
  samba:
    image: dperson/samba
    environment:
      TZ: 'Asia/Jakarta'
    networks:
      - default
    ports:
      - "137:137/udp"
      - "138:138/udp"
      - "139:139/tcp"
      - "445:445/tcp"
    read_only: true
    tmpfs:
      - /tmp
    restart: unless-stopped
    stdin_open: true
    tty: true
    volumes:
      - /mnt:/mnt:z
      - /mnt2:/mnt2:z
    command: '-s "Mount;/mnt" -s "Bobs Volume;/mnt2;yes;no;no;bob" -u "bob;bobspasswd" -p'
# share;path;browsable;ro;guest;users;admins;writelist;comment

networks:
  default:
```
