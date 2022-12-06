# Install WireGuard di Server VPS

sources:
- [https://golb.hplar.ch/2018/10/wireguard-on-amazon-lightsail.html](https://golb.hplar.ch/2018/10/wireguard-on-amazon-lightsail.html)

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

# edit nano /etc/sysctl.conf, hapus # didepan net.ipv4.ip_forward
```

# Setup WireGuard

```
cd /etc/wireguard/
umask 077 
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

nano /etc/wireguard/wg0.conf

[Interface]
Address = 192.168.2.1
PrivateKey = server_private_key
ListenPort = 54321
SaveConfig = false
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A INPUT -s 192.168.2.0/24 -p udp -m udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D INPUT -s 192.168.2.0/24 -p udp -m udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT

[Peer]
PublicKey = client_public_key
AllowedIPs = 192.168.2.2/32
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
nano client2
```

```
[Interface]
Address = 192.168.2.2/32
PrivateKey = client_private_key
DNS = 192.168.2.1
MTU = 1280

[Peer]
PublicKey = server_public_key
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
qrencode -t ansiutf8 < client2
```
![image](https://user-images.githubusercontent.com/89820226/205880161-da3bf4cc-4dc3-4d4f-9153-9cf00210f954.png)
client android
![image](https://user-images.githubusercontent.com/89820226/205880187-fc4bd8bc-ba4a-4428-958b-16c622aa2ebd.png)





