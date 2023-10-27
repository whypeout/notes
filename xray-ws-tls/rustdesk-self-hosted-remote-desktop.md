[https://rustdesk.com/docs/en/self-host/client-configuration/](https://rustdesk.com/docs/en/self-host/client-configuration/)
[https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/install/](https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/install/)
[https://medium.com/@gdadkisson/donovan-adkisson-how-to-solve-rustdesk-relayed-and-unencrypted-connection-issues-4f4de5fddf86](https://medium.com/@gdadkisson/donovan-adkisson-how-to-solve-rustdesk-relayed-and-unencrypted-connection-issues-4f4de5fddf86)
[https://www.vultr.com/docs/how-to-install-rustdesk-remote-desktop-server-on-ubuntu/](https://www.vultr.com/docs/how-to-install-rustdesk-remote-desktop-server-on-ubuntu/)

# Server

```
./hbbs -r 0.0.0.0 -k _
./hbbr -k _
```

Port used :
- hbbs 21115/tcp (nat type test), 21116/udp (id registration & heartbeat) 21116/tcp tcp holepunch & connection service, 21118/tcp (web client:disable)
- hbbr 21117/tcp (relay service), 21119/tcp (web client:disable)

---

# Client


- ID Server : internal.bosnetdis.com (hbbs.bosnetdis.com:21116) 103.54.218.250 192.168.0.233;
- Relay Server : internal.bosnetdis.com 103.54.218.250 192.168.0.233; hbbr (nginx:21117)
- API Server : (http::21114 Pro only)
- KEY : "Public KEY"

# Automatic

`{"host":"host-addr", "key": "public key", "api": "http://host-addr:21114"}`

# windows client

rename `rustdesk.exe` to `rustdesk-host=internal.bosnetdis.com,key=public-key=32.exe`


# KEY MISMATCH
key mismatch terjadi karena server menjalankan hbbs dan hbbr dg private/public key yg berbeda dg yg tersimpan di client untuk server tersebut.
untuk menghindari hal ini terjadi, jalankan server dg `hbbs -r 0.0.0.0 -k <public key>` dan `hbbr -k <public key>` .. public key bisa dilihat dari file `id_ed25519.pub`
pastikan public key yg sama dipakai di server dan client.
