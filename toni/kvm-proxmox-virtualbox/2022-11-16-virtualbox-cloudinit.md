```
## Install necessary packages
$ sudo apt-get install virtualbox-ose qemu-utils genisoimage cloud-utils

## get kvm unloaded so virtualbox can load
$ sudo modprobe -r kvm_amd kvm_intel
$ sudo service virtualbox stop
$ sudo service virtualbox start

## URL to most recent cloud image of 18.04
$ img_url="http://cloud-images.ubuntu.com/server/releases/18.04/release"
$ img_url="${img_url}/ubuntu-18.04-server-cloudimg-amd64-disk1.img"


## on precise, cloud-localds is not in archive. just download.
$ localds_url="http://bazaar.launchpad.net/~cloud-utils-dev/cloud-utils/trunk/download/head:/cloudlocalds-20120823015036-zkgo0cswqhhvener-1/cloud-localds"
$ which cloud-localds ||
  { sudo wget "$localds_url" -O /usr/local/bin/cloud-localds &&
    sudo chmod 755 /usr/local/bin/cloud-localds; }

## download a cloud image to run, and convert it to virtualbox 'vdi' format
$ img_dist="${img_url##*/}"
$ img_raw="${img_dist%.img}.raw"
$ my_disk1="my-disk1.vdi"
$ wget $img_url -O "$img_dist"
$ qemu-img convert -O raw "${img_dist}" "${img_raw}"
$ vboxmanage convertfromraw "$img_raw" "$my_disk1"

## create user-data file and a iso file with that user-data on it.
$ seed_iso="my-seed.iso"
$ cat > my-user-data <<EOF
#cloud-config
password: passw0rd
chpasswd: { expire: False }
ssh_pwauth: True
EOF
$ cloud-localds "$seed_iso" my-user-data


##
## create a virtual machine using vboxmanage
##
$ vmname="precise-nocloud-1"
$ vboxmanage createvm --name "$vmname" --register
$ vboxmanage modifyvm "$vmname" \
   --memory 512 --boot1 disk --acpi on \
   --nic1 nat --natpf1 "guestssh,tcp,,2222,,22"
## Another option for networking would be:
##   --nic1 bridged --bridgeadapter1 eth0
$ vboxmanage storagectl "$vmname" --name "IDE_0"  --add ide
$ vboxmanage storageattach "$vmname" \
    --storagectl "IDE_0" --port 0 --device 0 \
    --type hdd --medium "$my_disk1"
$ vboxmanage storageattach "$vmname" \
    --storagectl "IDE_0" --port 1 --device 0 \
    --type dvddrive --medium "$seed_iso"
$ vboxmanage modifyvm "$vmname" \
 --uart1 0x3F8 4 --uartmode1 server my.ttyS0

## start up the VM
$ vboxheadless --vnc --startvm "$vmname"

## You should be able to connect to the vnc port that vboxheadless
## showed was used.  The default would be '5900', so 'xvcviewer :5900'
## to connect.
##
## Also, after the system boots, you can ssh in with 'ubuntu:passw0rd'
## via 'ssh -p 2222 ubuntu@localhost'
##
## To see the serial console, where kernel output goes, you
## can use 'socat', like this:
##   socat UNIX:my.socket -

## vboxmanage controlvm "$vmname" poweroff
$ vboxmanage controlvm "$vmname" poweroff

## clean up after ourselves
$ vboxmanage storageattach "$vmname" \
   --storagectl "IDE_0" --port 0 --device 0 --medium none
$ vboxmanage storageattach "$vmname" \
   --storagectl "IDE_0" --port 1 --device 0 --medium none
$ vboxmanage closemedium dvd "${seed_iso}"
$ vboxmanage closemedium disk "${my_disk1}"
$ vboxmanage unregistervm $vmname --delete
``
                           

sources:
- [https://gist.github.com/smoser/6066204](https://gist.github.com/smoser/6066204)                           
