# Automation dg Proxmox dan Terraform

source:
- [yt https://www.youtube.com/watch?v=UXXIl421W8g](https://www.youtube.com/watch?v=UXXIl421W8g)
- [https://austinsnerdythings.com/2021/09/01/how-to-deploy-vms-in-proxmox-with-terraform/](https://austinsnerdythings.com/2021/09/01/how-to-deploy-vms-in-proxmox-with-terraform/)
 
# Tasklist

1. Install Terraform
2. Otentikasi Terraform ke Proxmox: User/Pass vs API
3. Terraform basic init & instalasi provider
4. Develop Terraform Plan
5. Terraform Plan
6. Run Terraform Plan & watch VM ready


# 1 Install Terraform

```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform
```

# 2 Otentikasi Terraform

1. Username/Password: gunakan user root dan password layaknya di web ui
2. API Keys: buat user baru, berikan user permission yg diperlukan dan setup API Key sebagai pengganti password

untuk menghindari kita menuliskan password (root) di file terraform, kita pakai API Key
![image](https://user-images.githubusercontent.com/89820226/221803765-69efa5f0-a987-44a5-a203-07f2987ba920.png)

lalu kita buat api token
![image](https://user-images.githubusercontent.com/89820226/221803955-4171ba1d-f5a5-4676-93a5-bb7ba8476d66.png)

kita copy&simpan token tadi ke .env/terraform files
![image](https://user-images.githubusercontent.com/89820226/221804097-d41a9def-7cc5-4a7e-95d0-96aba20d20ea.png)

lalu kita tambah role. Permission->Add->Path=/
![image](https://user-images.githubusercontent.com/89820226/221804273-1b71841c-48f0-4836-9029-086d315280a4.png)

kita juga perlu tambahkan permission ke storage yg akan kita pakai, misal local-zfs/local/local-lvm
![image](https://user-images.githubusercontent.com/89820226/221804535-e40fc767-748d-45d1-b524-5c0d5e7e1e90.png)

sekarang kita selesai dg permissioin:
![image](https://user-images.githubusercontent.com/89820226/221804615-2448c789-e596-4791-aa8e-88cd27f77387.png)

# 3 instalasi provider dan informasi dasar

ada 3 stage: init, plan & apply. kita mulai dg plans yg berupa konfigurasi file apa yg akan kita lakukan. buat 2 file (main.tf & vars.tf) didalam folder (terraform-blog)

```
cd ~
mkdir terraform-blog && cd terraform-blog
touch main.tf vars.tf
```

kita tambahkan provider proxmox

```
# nano main.tf
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.7.4"
    }
  }
}
```

# 4 lengkapi terraform plan

```

```




