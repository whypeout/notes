# Disable Netplan and Cloud Config

```
apt update
apt upgrade -y

# reinstall paket
apt install ifupdown

# edit konfig interface
nano /etc/network/interfaces
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

allow-hotplug enp0s3
auto enp0s3
iface enp0s3 inet static
address 192.168.200.246
netmask 255.255.255.0
broadcast 192.168.200.255
gateway 192.168.200.254
dns-nameservers 1.1.1.1 1.0.0.1
# save & close

# apply config tanpa reboot
ifdown --force enp0s3 lo && ifup -a
systemctl unmask networking
systemctl enable networking
systemctl restart networking

# disable & remove service tidak perlu
systemctl stop systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
systemctl disable systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
systemctl mask systemd-networkd.socket systemd-networkd networkd-dispatcher systemd-networkd-wait-online
apt-get -y purge nplan netplan.io
```

## Menyesuaikan dns resolver

secara default mulai ubuntu 18.04 menggunakan systemd-resolved sbg stub-resolver

```
# nano /etc/systemd/resolved.conf
DNS=1.1.1.1 1.0.0.1

# systemctl restart systemd-resolved
# rm /etc/resolve.conf
# ln -r /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## Menggunakan systemd-networkd utk mengatur ip

```
# nano /etc/systemd/network/50-enp0s3.network
[Match]
Name=enp0s3

[Network]
DHCP=no
Address=192.168.200.247/24
Gateway=192.168.200.254

# systemctl restart systemd-networkd
```

source:
- [https://tweenpath.net/ubuntu-18-04-disable-netplan-switch-networking-etc-network-interfaces/](https://tweenpath.net/ubuntu-18-04-disable-netplan-switch-networking-etc-network-interfaces/)


