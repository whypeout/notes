# Sharing Directory via SMB on Docker Container

sources:
- [http://www.freekb.net/Article?id=3428](http://www.freekb.net/Article?id=3428)


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




