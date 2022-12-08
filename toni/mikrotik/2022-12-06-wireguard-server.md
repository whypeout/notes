# Install WireGuard di Server VPS

sources:
- [https://golb.hplar.ch/2018/10/wireguard-on-amazon-lightsail.html](https://golb.hplar.ch/2018/10/wireguard-on-amazon-lightsail.html)
- [https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4](https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4)

# Setup VPS

```
sudo -i
apt update
apt upgrade -y
apt full-upgrade -y

echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
apt update
apt install wireguard unbound qrencode curl

# atau
sudo apt-add-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard

# edit nano /etc/sysctl.conf
# enable ip forward dg menghapus # didepan net.ipv4.ip_forward
```

# Setup WireGuard

```
cd /etc/wireguard/
umask 077 
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

# contoh output
example privatekey - mNb7OIIXTdgW4khM7OFlzJ+UPs7lmcWHV7xjPgakMkQ=
example publickey - 0qRWfQ2ihXSgzUbmHXQ70xOxDd7sZlgjqGSPA9PFuHg=

# tambahkan presharedkey jika diperlukan, baik di server maupun di client
# wg genpsk > preshared
```
```
nano /etc/wireguard/wg0.conf
##########
[Interface]
Address = 192.168.2.1/24
PrivateKey = server_private_key
ListenPort = 54321
SaveConfig = false
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A INPUT -s 192.168.2.0/24 -p udp -m udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D INPUT -s 192.168.2.0/24 -p udp -m udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT

[Peer]
# peer 1 => remote tunnel
PublicKey = client_public_key
AllowedIPs = 192.168.2.2/32

[Peer]
# peer 2 => site to site tunnel
PublicKey = site-to-site-public-key
AllowedIPs = 192.168.200.0/24
```
```
sed -i "s/server_private_key/$(sed 's:/:\\/:g' server_private.key)/" wg0.conf
sed -i "s/client_public_key/$(sed 's:/:\\/:g' client_public.key)/" wg0.conf
```
hasil akhir kurleb:
```
[Interface]
Address = 192.168.2.1
PrivateKey = cMgbJqIl6CuU6U6gpXu4TwUlJ+TnAgaSa6Dc8b5g1F8=
ListenPort = 54321
...

[Peer]
PublicKey = GXehejiGNxfOk5bEKECYgQg0nM9cu80BxPJap47s3QE=
AllowedIPs = 192.168.2.2/32
```

> iptables interface %i bisa digunakan utk me-refer ke `systemctl start wg-quick@wg0.service`

# Setup DNS server UNBOUND

[https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/)
```
curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
chown -R unbound:unbound /var/lib/unbound

cd /etc/unbound/unbound.conf.d
nano unbound_srv.conf
```
```
server:
  num-threads: 4
  verbosity: 1
  root-hints: "/var/lib/unbound/root.hints"

  interface: 0.0.0.0
  max-udp-size: 3072

  access-control: 0.0.0.0/0        refuse
  access-control: 127.0.0.1        allow
  access-control: 192.168.2.0/24   allow

  private-address: 192.168.2.0/24

  hide-identity: yes
  hide-version: yes

  harden-glue: yes
  harden-dnssec-stripped: yes
  harden-referral-path: yes

  unwanted-reply-threshold: 10000000

  val-log-level: 1
  cache-min-ttl: 1800 
  cache-max-ttl: 14400
  prefetch: yes
  prefetch-key: yes 
```
```
systemctl restart unbound
systemctl enable unbound
systemctl status unbound
reboot
```
start wireguard
```
wg-quick up wg0
systemctl enable wg-quick@wg0.service
ifconfig wg0
```

# Client Setup

kita buat config utk client2

```
cd /etc/wireguard/
nano client2.conf
```

```
########### peer client remote vpn
[Interface]
Address = 192.168.2.2/32
PrivateKey = client_private_key
DNS = 192.168.2.1
MTU = 1280

[Peer]
PublicKey = server_public_key
PresharedKey = [presharedkey-jika-ada]
Endpoint = 54.147.249.172:54321
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
```
sed -i "s/client_private_key/$(sed 's:/:\\/:g' client_private.key)/" client2
sed -i "s/server_public_key/$(sed 's:/:\\/:g' server_public.key)/" client2
```
generate qrcode utk di scan di client
```
qrencode -t ansiutf8 < client2.conf
```
![image](https://user-images.githubusercontent.com/89820226/205880161-da3bf4cc-4dc3-4d4f-9153-9cf00210f954.png)
client android
![image](https://user-images.githubusercontent.com/89820226/205880187-fc4bd8bc-ba4a-4428-958b-16c622aa2ebd.png)


# Peer Site to Site tunnel

```
########## peer site to site tunnel
[Interface]
Address = 192.168.2.3/32
PrivateKey = client_private_key
DNS = 192.168.2.1
MTU = 1280

[Peer]
PublicKey = server_public_key
PresharedKey = [presharedkey-jika-ada]
Endpoint = 54.147.249.172:54321
AllowedIPs = 192.168.0.0/24 # network dibelakang server/vps
PersistentKeepalive = 25
```

# client connect & check

```
sudo wg show
```

# Server command

setiap menambahkan peer, kita perlu me-restart service & interface wireguard agar peer baru bisa terkoneksi

```
# Start stop interface
wg-quick up wg0
wg-quick down wg0

# Start stop services
sudo systemctl stop wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```

alternativ agar tidak perlu me-restart service utk menambahkan client, gunakan wg-tool

```
# Add Peer
wg set wg0 peer <client-publickey> allowed-ips 192.168.2.5/32

# Melihat koneksi
wg

# save ke config
wg-quick save wg0






