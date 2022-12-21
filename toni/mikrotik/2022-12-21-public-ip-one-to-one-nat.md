# Link IP Public ke IP Local dg One-to-One NAT

```
/ip address
add address=43.254.126.211/29 interface=ether5
add address=43.254.126.210/29 interface=ether5
add address=43.254.126.212/29 interface=ether5
add address=192.168.10.1/24 interface=ether1

/ip firewall nat
add chain=src-nat action=src-nat src-address=192.168.10.2/32 to-address=43.254.126.210
add chain=src-nat action=src-nat src-address=192.168.10.3/32 to-address=43.254.126.212
add chain=dst-nat action=dst-nat dst-address=43.254.126.210/32 to-address=192.168.10.2
add chain=dst-nat action=dst-nat dst-address=43.254.126.212/32 to-address=192.168.10.3
add chain=src-nat action=src-nat src-address=192.168.10.0/24 to-address=43.254.126.211
```

sources:
- [https://wiki.mikrotik.com/wiki/How_to_link_Public_addresses_to_Local_ones](https://wiki.mikrotik.com/wiki/How_to_link_Public_addresses_to_Local_ones)
- [https://blog.tayabkhan.com/2015/12/nat-with-mikrotik-one-to-one-mapping.html](https://blog.tayabkhan.com/2015/12/nat-with-mikrotik-one-to-one-mapping.html)
