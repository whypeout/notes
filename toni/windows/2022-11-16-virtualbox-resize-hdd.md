# 1. Convert VMDK to VDI

jika kita import OVA file, akan berakhir dg VMDK disk image. pertama, detach image/hdd dari VM

![detach](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30-14_22_10-Oracle-VM-VirtualBox-Manager-300x173.png)

clone vmdk ke vdi (sekaligus utk backup)

jangan lupa remove vmdk dari Virtual Machine Manager, sebelum menambahkan vdi ke Virtual Machine Manager

```
c:\program files\oracle\virtualbox\>
vboxmanage.exe clonemedium --format vdi d:\namavm\hasil-import.vmdk d:\namavm\hasil-convert.vdi
```

# 2. Resize VDI Disk Image

tambahkan dari 10GB ke 20000MB (20GB)

```
# vbox 6.0+
vboxmanage.exe modifymedium hasil-convert.vdi --resize 20000
# vbox 5.2
vboxmanage.exe modifyhd hasil-convert.vdi --size 20000
```

# 3. Boot into Ubuntu Live CD

Resize Disk Size

![attach](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-32-03.png)

![boot](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-32-44-300x166.png)

![try](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-34-03-try-ubuntu-300x194.png)

![gparted](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-36-12-gparted-208x300.png)

![deactive](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-37-21-deactivate-300x177.png)

![extend](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-38-09-resize-the-extended-partition-300x197.png)

![max](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-38-58-resize-the-disk-300x239.png)

![check&apply](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-39-39-check-the-disk-300x204.png)

# 4. PV resize on ubuntu

```
sudo pvresize /dev/sda5
sudo lvresize -l +100%FREE /dev/mapper/ubuntu-vg-root
sudo e2fsck -f /dev/mapper/ubuntu-vg-root
sudo resize2fs /dev/mapper/ubuntu-vg-root
sudo reboot
```

![pvresize](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-47-05-pvresize-300x26.png)
![lvresize](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-48-28-lvresize-300x27.png)
![restart](https://technology.amis.nl/wp-content/uploads/2017/01/2017-01-30_14-54-41-result-300x183.png)

# 5. Small Resize

bersihkan Guest OS dg BleachBit utk menghapus temporary files. atau CCleaner di windows.

```
sudo telinit 1
mount -o remount,ro /dev/sda1
zerofree -v /dev/sda1
# ubah /etc/fstab utk /dev/mapper/ubuntu-vg-root options menjadi RO
reboot

sudo -i
telinit 1
zerofree /dev/mapper/ubuntu-vg-root
mount -o remount,rw /dev/mapper/ubuntu-vg-root
# update fstab dg options aslinya. reboot, sekarang freespace sudah zero

# di host os:
vboxmanage.exe modifymedium --compact hasil-convert.vdi
```

sources:
- [https://technology.amis.nl/platform/virtualization-and-oracle-vm/ubuntu-vm-virtualbox-increase-size-disk-make-smaller-exports-distribution/](https://technology.amis.nl/platform/virtualization-and-oracle-vm/ubuntu-vm-virtualbox-increase-size-disk-make-smaller-exports-distribution/)



