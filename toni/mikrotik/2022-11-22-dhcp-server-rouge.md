# Mencegah Multiple DHCP Server pada MikroTik

sources:
- [https://labkom.co.id/mikrotik/mencegah-multiple-dhcp-server-pada-mikrotik](https://labkom.co.id/mikrotik/mencegah-multiple-dhcp-server-pada-mikrotik)
- [https://dhoniedasta.wordpress.com/2013/06/02/79/](https://dhoniedasta.wordpress.com/2013/06/02/79/)
- [https://citraweb.com/artikel_lihat.php?id=210](https://citraweb.com/artikel_lihat.php?id=210)

```
/interface bridge filter
# LAN Interface
add action=accept chain=forward dst-port=67 ip-protocol=udp mac-protocol=ip out-interface=ether2 src-port=68
add action=drop chain=forward dst-port=67 ip-protocol=udp mac-protocol=ip src-port=68

# enable dhcp alert
/ip dhcp-server set authoritative=yes
```

![img](https://labkom.co.id/wp-content/uploads/2020/04/brigde-rule.png)

![img](https://labkom.co.id/wp-content/uploads/2020/04/dhcp-alert.png)

kirim alert via telegram bot

```
:local output
:local bot "BOT";
:local chatID "CHAT";
:local mikrotik [/system identity get name] ;
:foreach activeIndex in=[/ip dhcp-server alert find comment=lan1] do={
:local interface [/ip dhcp-server alert get value-name="interface" $activeIndex];
:local valid [/ip dhcp-server alert get value-name="valid" $activeIndex];
:local unk [/ip dhcp-server alert get value-name="unknown-server" $activeIndex];
:log warning "Nama Mikrotik: $mikrotikNama interface: $interface \nMAC VALID: $valid \nMAC ROUGE$unk";
/tool fetch url="https://api.telegram.org/bot$bot/sendmessage\?chat_id=$chatID&text=\E2\9C\85 ROTER: $mikrotik %0A\E2\9C\85 Interface: $interface %0A\E2\9C\85 Valid MAC: $valid %0A\E2\9C\85 MAC Rouge: $unk" keep-result=no
}
```

Interface ARP Reply-Only

![img](https://dhoniedasta.files.wordpress.com/2011/05/dhcponly.png)

DHCP Server :: Add ARP for Leases

![img](https://dhoniedasta.files.wordpress.com/2011/05/dhcpaddarp-1.png)




