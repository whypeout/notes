# Menambahkan Static Route pada Interface VPN di Windows 

source: [Automatically Add Static Routes After Connecting to VPN](http://woshub.com/add-routes-after-connect-vpn-windows/)

Buka Powershell as Administrator

`Get-VpnConnection`

![getvpn](http://woshub.com/wp-content/uploads/2021/09/powershell-get-vpnconnection-list-on-windows-10.png)

Matikan default gateway 

`Set-VpnConnection –Name workVPN -SplitTunneling $True`

```pwsh
Add-VpnConnectionRoute -ConnectionName workVPN -DestinationPrefix 192.168.11.0/24 –PassThru
Add-VpnConnectionRoute -ConnectionName workVPN -DestinationPrefix 10.1.0.0/16 –PassThru
```

![add-route](http://woshub.com/wp-content/uploads/2021/09/add-vpnconnectionroute-adding-route-automatically.png)

```pwsh
DestinationPrefix : 192.168.11.0/24
InterfaceIndex :
InterfaceAlias : workVPN
AddressFamily : IPv4
NextHop : 0.0.0.0
Publish : 0
RouteMetric : 1
```

![pwsh](http://woshub.com/wp-content/uploads/2021/09/show-vpn-connection-custom-route.png)

```pwsh
Get-NetRoute : No MSFT_NetRoute objects found with property 'DestinationPrefix' equal to '192.168.11.0/24'. Verify the value of the property and retry. CmdletizationQuery_NotFound_DestinationPrefix,Get-NetRoute
```

![pwsh](http://woshub.com/wp-content/uploads/2021/09/vpn-route-is-automatically-deleted-after-disconnec.png)

Remove Static Routing for VPN

```pwsh
Remove-VpnConnectionRoute -ConnectionName workVPN -DestinationPrefix 192.168.111.0/24 -PassThru
```

Buat script dan task schedule utk menambahkan route

```cmd
interface ipv4
add route prefix=192.168.11.24 interface="workVPN" store=active
add route prefix=10.1.0.0/16 interface="workVPN" store=active
exit
```

```pwsh
schtasks /create /F /TN "Add VPN routes" /TR "netsh -f C:\PS\vpn_route.netsh" /SC ONEVENT /EC Application /RL HIGHEST /MO "*[System[(Level=4 or Level=0) and (EventID=20225)]] and *[EventData[Data='My VPN']]"
```



