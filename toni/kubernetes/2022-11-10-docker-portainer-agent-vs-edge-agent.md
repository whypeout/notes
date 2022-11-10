# Menambahkan Instance ke Portainer Server sebagai Agent atau Edge ?

Edge Agent dibuat utk memanage Edge Compute env dimana device tidak bisa membuka port
seperti pada Portainer Agent biasa.

## Prep

### Expose Tunnel Port

Edge Agent berkomunikasi dg Portainer Server via Port 8000 (atau 30776 pada Kubernetes NodePort).
lewat port ini, Edge Agent bisa menarik Portainer instance, connect ke Portainer, melihat jika perlu,
kemunian memulai tunnel/menerima config update. Tanpa tunnel port terbuka disisi Portainer Server, kita tidak
bisa mengakses Edge endpoint. Jika kita sudah men-deploy Portainer tanpa membuka port ini,
kita perlu mendeploy ulang Portainer dg membuka port ini. opsi CLI `--tunnel-port` utk mengatur port berbeda
jika port tersebut sudah dipakai.

### Metode Deployment

- Portainer dg TLS : Jika Portainer menggunakan TLS, Agent akan melakukan koneksi via HTTPS ke Portainer (Recommended).
- Portainer dg Self-Signed Certs : Jika Portainer menggunakan Self Signed Certificates, Edge Agent harus di deploy dg opsi `-e EDGE_INSECURE_POLL=1`.
  tanpa flag ini, edge tidak bisa komunikasi dg Portainer.
- Portainer Fallback to HTTP : jika portainer tidak menggunakan konfig diatas, akan menggunakan HTTP utk agent polling sbg fallback.
  Not Recommended karena tidak secure.
  
## Menambahkan Edge Env ke Portainer

Buka Portainer, Masuk Menu **Environment**, kemudian **Add Environment**

![add-env](https://2914113074-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FUeb1lPrWdn7TlqnXoAAC%2Fuploads%2FCN9PzdDVV5WbVUfQ35Hi%2F2.16-environments-add.gif?alt=media&token=4b9a1b8a-38a1-4e97-8b83-f3d7e906f450)

Pilih Docker / Kubernetes tergantung Environment yg digunakan. klik **Start Wizard**.
kemudian pilih **Edge Agent**. masukkan details env sbb:
- Name : Nama Env, Misal KubeLokal
- Portainer Server URL : URL dan Port dari Portainer Server sbgmn terlihat di Edge Env.
  gunakan FQDN jika DNS (lokal) tersedia.
![fqdn](https://2914113074-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FUeb1lPrWdn7TlqnXoAAC%2Fuploads%2Fu263JgzO29fvsS7nWLZ9%2F2.15-settings-env-addenv-edge-name.png?alt=media&token=05bdd712-44b8-44cb-b86b-c86047349872)

sbg Opsional Step, expand **More Settings** dan sesuaikan Poll freq - mengatur berapa sering Edge Agent
akan mengecek Portainer Server utk new jobs. default every 5 seconds.
kita bisa mengkategorikan env dg menambahkan **group** atau **tag**
klik **Create** setelah selesai konfig


