Buat Konfig di situs penyedia server gratis [https://www.fastssh.com/](https://www.fastssh.com/)

```
# vmess 443 tls/ssl
vmess://eyJhZGQiOiJ0c2VsLm1lIiwiYWlkIjoiMCIsImlkIjoiY2MzYjNhOTAtZDM5MS0xMWVkLWI0MzItMjA1YzZkNWY1ZDc4IiwiaG9zdCI6InNnLTguMHJkLm5ldCIsIm5ldCI6IndzIiwicGF0aCI6Ii9oeWtqN3psdiIsInBvcnQiOiI0NDMiLCJwcyI6IlI0NDM6Vm1lc3MgU2luZ2Fwb3JlIE1lbGJpIDIgMjAyMy0wNC0xMiIsInRscyI6InRscyIsInR5cGUiOiJub25lIiwidiI6IjIifQ==

# vmess 80 ws
vmess://eyJhZGQiOiJ0c2VsLm1lIiwiYWlkIjoiMCIsImlkIjoiY2MzYjNhOTAtZDM5MS0xMWVkLWI0MzItMjA1YzZkNWY1ZDc4IiwiaG9zdCI6InNnLTguMHJkLm5ldCIsIm5ldCI6IndzIiwicGF0aCI6Ii9oeWtqN3psdiIsInBvcnQiOiI4MCIsInBzIjoiUjgwOlZtZXNzIFNpbmdhcG9yZSBNZWxiaSAyIDIwMjMtMDQtMTIiLCJ0bHMiOiIiLCJ0eXBlIjoibm9uZSIsInYiOiIyIn0=
```

howdy trojan go vpn

```
#trojan go/gfw
trojan://5f9d48f0-d454-11ed-8ad7-1239d0255272@id-1.test3.net:443/ruangguru.com?sni=ruangguru.com

#igniter go
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1234,
    "remote_addr": "ruangguru.com",
    "remote_port": 443,
    "dns": [
        "1.1.1.1"
    ],
    "password": [
        "5f9d48f0-d454-11ed-8ad7-1239d0255272"
    ],
    "ssl": {
        "sni": "id-1.test3.net"
    },
    "websocket": {
        "enabled": true,
        "path": "\/howdy",
        "hostname": "id-1.test3.net"
    }
}
#sager anxray
trojan-go://5f9d48f0-d454-11ed-8ad7-1239d0255272@ruangguru.com:443/?sni=id-1.test3.net&type=ws&host=id-1.test3.net&path=%2Fhowdy

#trojan go v2rayng
trojan://5f9d48f0-d454-11ed-8ad7-1239d0255272@ruangguru.com:443/?sni=id-1.test3.net&type=ws&host=id-1.test3.net&path=%2Fhowdy
```
----

V2Ray Install Tutorial
- [official](https://www.v2ray.com/en/welcome/install.html)
- [github v2ray core releases](https://github.com/v2ray/v2ray-core/releases)
- [github v2ray dist](https://github.com/v2ray/dist)
- [github v2ray docker --updated 3ya](https://github.com/v2fly/docker)
- [hub docker v2ray](https://hub.docker.com/r/v2ray/official/)
- [v2fly core](https://hub.docker.com/r/v2fly/v2fly-core)
- [hxp v2ray : nginx v2ray](https://hxp.plus/2020/02/07/v2ray/)
- [official guide v2fly : websocket+tls+web](https://guide.v2fly.org/en_US/advanced/wss_and_web.html)
- [official guide v2fly: websocket](https://guide.v2fly.org/en_US/advanced/websocket.html)
- [gist: nginx vless vmess port 443 SSL](https://gist.github.com/megrxu/1ad492eac2d343b4dbd4bb964ca37670)
- [X-UI : Xray UI for create inbound account](https://github.com/vaxilu/x-ui/)
- [x-ui local laptop](http://192.168.0.36:12345/xui/inbounds)
- Clash support Wireguard & OpenConnect VPN spt: PaloAlto GlobalProtect, Cisco AnyConnect, FortiGate [https://github.com/Dreamacro/clash/wiki/Examples](https://github.com/Dreamacro/clash/wiki/Examples)
- [Official OpenClash Core](https://github.com/Dreamacro/clash)
- [!! acme.sh SSL LetsEncrypt with DNS CloudFlare ZoneAPI Wildcard DNS](https://www.cyberciti.biz/faq/issue-lets-encrypt-wildcard-certificate-with-acme-sh-and-cloudflare-dns/), include: ssh hook, pre-hook&post-hook ke nodes lain, renew script via cronjob, install cert&ecc to nginx
- [acme.sh github repo](https://github.com/acmesh-official/acme.sh), [acme.sh github wiki](https://github.com/acmesh-official/acme.sh/wiki)

Install di linux



```
curl -Ls https://install.direct/go.sh | sudo bash
```
installed: 
- /usr/bin/v2ray/v2ray : v2ray executable
- /usr/bin/v2ray/v2ctl : utility
- /etc/v2ray/config.json : config file
- /usr/bin/v2ray/geoip.dat : ip data file
- /usr/bin/v2ray/geosite.dat : domain data file

service:
- /etc/systemd/system/v2ray.service : systemd
- /etc/init.d/v2ray : sysv

after instal:
- /etc/v2ray/config.json update sesuai skenario
- `service v2ray start` or `systemctl start v2ray`

Install nginx

```
yum install nginx
apt install nginx
```
update config v2ray, generator [https://www.veekxt.com/utils/v2ray_gen](https://www.veekxt.com/utils/v2ray_gen)
```
nano /etc/v2ray/config.json
{
  "inbounds": [
    {
      "port": 10000,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "cd6e4579-7a1f-4784-91dd-e0e9e788c7ee",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
systemctl restart v2ray
systemctl status v2ray
systemctl enable v2ray
```

Install acme.sh ssl cert dg letsencrypt

```
apt install nginx socat

curl https://get.acme.sh | sh
./acme.sh --install --home /etc/acmesh --config-home /etc/ssl/data --cert-home /etc/ssl/certs --accountemail "user@domain"
acme.sh --register-account -m "user@domain"

mkdir /etc/pki/nginx/
mkdir /etc/pki/nginx/private/

systemctl stop nginx

# standalone
sudo ~/.acme.sh/acme.sh --issue -d yourdomain.com --standalone -k ec-256

# renew
sudo ~/.acme.sh/acme.sh --renew -d yourdamain.com --force --ecc

# install to path
sudo ~/.acme.sh/acme.sh --installcert -d yourdomain.com --fullchainpath /etc/pki/nginx/server.crt --keypath /etc/pki/nginx/private/server.key --ecc
```
update /etc/nginx/nginx.conf
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  yourdomain.com;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
		 proxy_pass         https://dev-hxp.github.io;
		 proxy_redirect     off;
		 proxy_set_header   User-Agent "Mozilla/5.0";
		 proxy_set_header   Host                        $host;
		 proxy_set_header   X-Real-IP                $remote_addr;
		 proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.

    server {
        listen       443 ssl;
	ssl on;
        server_name  yourdomain.com;
        ssl_certificate "/etc/pki/nginx/server.crt";
        ssl_certificate_key "/etc/pki/nginx/private/server.key";
	ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers           HIGH:!aNULL:!MD5;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
	
	location /ray {
        	 proxy_redirect off;
        	 proxy_pass http://127.0.0.1:10000;
       		 proxy_http_version 1.1;
        	 proxy_set_header Upgrade $http_upgrade;
        	 proxy_set_header Connection "upgrade";
        	 proxy_set_header Host $http_host;

        	 # Show realip in v2ray access.log
        	 proxy_set_header X-Real-IP $remote_addr;
        	 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location / {
		 proxy_pass         https://dev-hxp.github.io;
		 proxy_redirect     off;
		 proxy_set_header   User-Agent "Mozilla/5.0";
		 proxy_set_header   Host                        $host;
		 proxy_set_header   X-Real-IP                $remote_addr;
		 proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```
```
systemctl restart nginx
systemctl enable nginx
```

ada 3 hal utk diperhatikan:
- konek v2ray lewat port 443, bukan 10000
- enable TLS di v2ray (client)
- enable websocket dan path websocket harus `/ray` sesuai konfig nginx & config.json tadi

----

- OpenClash Android Config YAML [https://globalssh.us/clash-android-openclash-example-config-yaml/](https://globalssh.us/clash-android-openclash-example-config-yaml/)
- [OpenClash Config Convert](https://globalssh.us/clash-config-generator/) : Convert VMESS VLESS Trojan ke CLASH Format
- [online v2ray config generator](https://intmainreturn0.com/v2ray-config-gen/)
- [v2ray config generator with docker](https://github.com/SonyaCore/V2RayGen)
- [https://github.com/SonyaCore/V2RayGen](https://github.com/SonyaCore/V2RayGen)
- [online uuid generator](https://www.uuidgenerator.net/)


----

dashboard cloudflare DNS only / proxied dns

```
A   eargon  45.127.133.228 DNS Only
A   emora   103.54.218.250 Proxied
CNAME ebiz  719f064e5900.sn.mynetname.net DNS Only
CNAME ebiznet 719f064e5900.sn.mynetname.net Proxied

```




## Side Notes

- HTML5 Game utk decoy TLS WS
- [https://github.com/TomMalbran/games/blob/gh-pages/index.html](https://github.com/TomMalbran/games/blob/gh-pages/index.html)
- [https://github.com/dmcinnes/HTML5-Asteroids](https://github.com/dmcinnes/HTML5-Asteroids)
- [https://github.com/mumuy/pacman](https://github.com/mumuy/pacman)

- phpvirtualbox [https://us.informatiweb.net/tutorials/it/virtualization/virtualbox-manage-your-vms-from-a-web-browser.html](https://us.informatiweb.net/tutorials/it/virtualization/virtualbox-manage-your-vms-from-a-web-browser.html)
- localhost bdi laptop [http://localhost:88/phpvirtualbox/](http://localhost:88/phpvirtualbox/)
- [https://wiki.archlinux.org/title/PhpVirtualBox](https://wiki.archlinux.org/title/PhpVirtualBox)
- official issue #279 support vbox 6.1.22 (6.1.42) [https://github.com/phpvirtualbox/phpvirtualbox/issues/279](https://github.com/phpvirtualbox/phpvirtualbox/issues/279)
- [SpaceDesk](https://www.spacedesk.net/#download) Android Jadi Monitor Kedua, Install Server di windows, dari android install client/viewer

- [ssh over websocket ws](https://blog.fastssh.com/vpn-protocol/how-to-use-ssh-over-websocket-cdn-for-free-internet/)
- [trojan go over websocket ws](https://blog.fastssh.com/tutorial/how-to-use-trojan-go-over-websocket-cdn-for-free-internet/)



----

- [Traefik : Apache Guacamole (path /guacamole & /websocket)](https://community.traefik.io/t/apache-guacamole-behind-traefik/11841)
- [Traefik: Websocket upgrade ws to wss](https://community.traefik.io/t/websocket-does-not-upgrade-ws-to-wss/12017)

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.guacamole.rule=Host(`remote.domain.xx`)"
  - "traefik.http.routers.guacamole.tls=true"
  - "traefik.http.middlewares.guacamole-prefix.addprefix.prefix=/guacamole" 
  - "traefik.http.routers.guacamole.middlewares=guacamole-prefix"

#or
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.guacamole.rule=Host(`remote.domain.xx`)"
  - "traefik.http.routers.guacamole.tls=true"
  - "traefik.http.middlewares.guacamole-prefix.addprefix.prefix=/guacamole"
  - "traefik.http.middlewares.guacamole-sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
  - "traefik.http.routers.guacamole.middlewares=guacamole-prefix,guacamole-sslheader"

# redirect all http to https
- traefik.http.routers.guacamole.rule=hostregexp(`{host:.+}`)
- traefik.http.routers.guacamole.entrypoints=web
- traefik.http.routers.guacamole.middlewares=redirect-to-https
- traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https
#Finally found an error - wrong prefix in the middleware config, "/guacamole/" instead of "/guacamole". 

# traefik ws to wss upgrade
Guys, the solution to the websocket problem is actually here, with two modifications.

The first: The middlewares must be created for safe route(443). I was using middlewares on the unsafe route(80).

The second: - traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https,wss

adding wss to the end.

# traefik 2.0 websocket
- traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto = https

```
