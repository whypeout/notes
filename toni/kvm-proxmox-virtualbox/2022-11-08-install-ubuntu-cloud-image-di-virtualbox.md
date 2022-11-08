# Install Ubuntu 20.04 | 22.04 Cloud Image (Minimal) on VirtualBox

sources: [https://www.how2shout.com/linux/install-ubuntu-20-04-22-04-cloud-image-minimal-on-virtualbox/](https://www.how2shout.com/linux/install-ubuntu-20-04-22-04-cloud-image-minimal-on-virtualbox/)

1. Download Ubuntu 20.04/22.04 Cloud Image

- [ubuntu focal 20.04](https://cloud-images.ubuntu.com/focal/current/)
- [ubuntu jammy 22.04](https://cloud-images.ubuntu.com/jammy/current/)

![download](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Download-Ubuntu-20.04-Cloud-Image.png)

2. Import Appliance in VirtualBox

- buka VirtualBox
- klik menu File
- pilih Import Appliance
- klik pada Folder Icon
  ![virtualbox](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Ubuntu-20.04-Cloud-Image-in-Virtual-machine.png)
  ![folder-icon](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Import-Appliance-in-VirtualBox.png)
- klik Import
  ![import](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Import-Ubuntu-20.04-Minimal-virtual-appliance.png)

3. Download Seed.iso

secara default, kita tidak bisa login ke minimal ubuntu server ini, karena **belum ada password** nya.
kita pakai **seed.iso** dari aws yg sudah berisi username dan password pada file `user-data` menggunakan **#cloud-config**
dan hostname pada file `meta-data`. [aws seed.iso download link](https://cdn.amazonlinux.com/os-images/2.0.20190612/)
![images](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Download-Seed-ISO-sample-file.png)

4. Insert Seed.ISO ke Virtualbox Ubuntu 20.04 LTS VM

edit Imported VM, hit Settings

![imported](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Open-VirtualBOx-virtual-machine-settings.png)

ke tab Storage, tambahkan Controller IDE jika belum ada

![cd iso](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Add-ISO-file-in-Virtualbox.png)

Tambahkan CD, cari dan pilih Seed.iso

![seed iso](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Add-Seed-ISO-file-for-Ubuntu-20.04-in-VirtualBox.png)

close dengan klik OK

![save](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Seed-ISO-file-in-Virtualbox.png)

5. Start Ubuntu Virtual Machine

![start](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Start-eh-virtual-machine.png)

6. Login dan Buat User Password Baru

default login Seed.iso username: user1 dan password: amazon

```sh
# set root password
sudo -i
passwd root

# tambahkan user dg akses sudo
adduser minibuntu
passwd minibuntu
usermod -aG sudo minibuntu

# ubah hostname
hostnamectl set-hostname minibuntu
echo '127.0.0.1 minibuntu' >> /etc/hosts
```

7. Remove disk Optical Devices (secara paksa)

![remove](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Remove-Virtual-DIsk.jpg)
![paksa](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Force-Unmount-Seed-ISO.png)

8. Restart virtual machine

setelah kita remove CD Seed.iso, kita restart virtual machine

```sh
sudo reboot
```

![login](https://www.how2shout.com/linux/wp-content/uploads/2021/11/Install-Ubuntu-20.04-or-22.04-minimal-Image-cloud-on-VirtualBox.png)

login ulang dg user baru yg kita buat tadi, minibuntu

