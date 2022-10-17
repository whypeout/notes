# Installing multi-node Kubernetes cluster with ubuntu Multipass

windows 10 vcredist upto 2020 [https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#visual-studio-2015-2017-2019-and-2022](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#visual-studio-2015-2017-2019-and-2022)

setup windows 10 multipass with ubuntu in virtualbox [https://multipass.run/docs/set-up-the-driver#heading--windows-use-virtualbox](https://multipass.run/docs/set-up-the-driver#heading--windows-use-virtualbox)

download virtualbox [latest](https://www.virtualbox.org/wiki/Downloads),
[windows](https://download.virtualbox.org/virtualbox/7.0.0/VirtualBox-7.0.0-153978-Win.exe),
[ext.pack](https://download.virtualbox.org/virtualbox/7.0.0/VirtualBoxSDK-7.0.0-153978.zip),
[canonical](https://github.com/canonical/multipass),
[linux virtual machine with multipass](https://medium.com/codex/use-linux-virtual-machines-with-multipass-4e2b620cc6),
[pstools.zip](https://download.sysinternals.com/files/PSTools.zip),
[multipass - change default driver to virtualbox](https://multipass.run/docs/set-up-the-driver#heading--windows-use-virtualbox)


### ubah driver bawaan windows dari hyperv ke virtualbox

install virtualbox latest (v7.0.0) dan install vc redist,  

buka virtualbox sbg user multipass

```
PS> & $env:USERPROFILE\Downloads\PSTools\PsExec.exe -s -i $env:VBOX_MSI_INSTALL_PATH\VirtualBox.exe
```

vboxmanage via cli

```
PS> & $env:USERPROFILE\Downloads\PSTools\PsExec.exe -s $env:VBOX_MSI_INSTALL_PATH\VBoxManage.exe list vms
```

port forwarding pada virtualbox

```
PS> & $env:USERPROFILE\Downloads\PSTools\PsExec.exe -s $env:VBOX_MSI_INSTALL_PATH\VBoxManage.exe controlvm "primary" natpf1 "myservice,tcp,,8080,,8081"
```

```
multipass set local.driver=virtualbox

multipass find
- docker jellyfin minikube appliance:openhab,plexmediaserver,adguard-home,mosquitto,nextcloud 18.04 20.04 22.04 

multipass list
node10 node11

multipass shell node10

multipass start node10
multipass stop node11
multipass restart node10
multipass suspend node11
multipass networks
multipass info node10
multipass exec node10 -- lsb_release -a

multipass launch -c 2 -d 50G -m 4G -n node10
multipass launch -c 2 -d 50G -m 4G -n node11
multipass launch -c 2 -d 50G -m 4G -n node12
multipass info --all
# info semua instance
```

install kubernetes, kubeadm, kubectl dan docker di ubuntu  
source [https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

install docker sources [https://docs.docker.com/engine/install/#server](https://docs.docker.com/engine/install/#server),
[linux install](https://docs.docker.com/desktop/install/linux-install/),
[docker desktop ubuntu](https://docs.docker.com/desktop/install/ubuntu/),
[download docker deb](https://desktop.docker.com/linux/main/amd64/docker-desktop-4.12.0-amd64.deb), or use 
[docker engine on ubuntu](https://docs.docker.com/engine/install/ubuntu/)

```
modprobe kvm
modprobe kvm_intel  #intel proc
modprobe kvm_amd    #amd proc
kvm-ok
lsmod | grep kvm
ls -la /dev/kvm
sudo usermod -aG kvm  $USER
```

```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux $(lsb_release -cs) stable" | sudo tee -a /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose

apt-cache madison docker-ce
apt install docker-ce=<version> docker-ce-cli=<version>
systemctl start docker #service docker start
sudo usermod -aG docker $USER
exit
docker run hello-world
#install with script:
curl -fsSL https://get.docker.com -o get-docker.sh
DRY_RUN=1 sh ./get-docker.sh
```
install CRI - container runtime interface - [adapter docker for kubernetes](https://github.com/Mirantis/cri-dockerd)

```
sudo -i
wget https://storage.googleapis.com/golang/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

install [MCR Mirantis Container Runtime on ubuntu](https://docs.mirantis.com/mcr/20.10/install/mcr-linux/ubuntu.html), or [MCR on Windows Servers](https://docs.mirantis.com/mcr/20.10/install/mcr-windows.html)

```
apt update
apt install apt-transport-https software-properties-common curl ca-certificates
export DOCKER_EE_URL="https://repos.mirantis.com"
export DOCKER_EE_VERSION=20.10
curl -fsSL "${DOCKER_EE_URL}/ubuntu/gpg" | sudo apt-key add -
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] $DOCKER_EE_URL/ubuntu $(lsb_release -cs) stable-$DOCKER_EE_VERSION"
apt update
apt install docker-ee docker-ee-cli docker-ee-rootless-extras containerd.io #MCR20.10.12 later
apt install docker-ee docker-ee-cli containerd.io #MCR10.10.11 before
```

[setup network 'calico'](https://mustafaakin.dev/posts/2020-04-11-installing-a-multi-node-kubernetes-cluster-with-ubuntu-multipass/)
with my home net 192.168.3.1/24, and multipass network 172.16.0.0/12.

```
kubeadm init --pod-network-cidr=172.16.0.0/12
# To start using your cluster, run as reguler user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
# You should now deploy a pod network to the cluster.
# Run "kubectl apply -f [podnetwork].yaml
https://kubernetes.io/docs/concepts/cluster-administration/addons/
# Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 10.161.139.229:6443 --token ahdsflahdsflahsdlkfhadl --discovery-token-ca-cert-hash sha256:adfalkdhsfklasdkfjaskldjfalkdsjfklajsd

# setelah kubectl join di masing2 node2 node3 `kubeadm join` hasil dari perintah `kubeadm init` di node pertama. nodes akan pull Docker images and kubelets will join the cluster really quick.

kubectl get node
NAME  STATUS    ROLES     AGE     VERSION
node1 NotReady  master    5m42s   v1.18.1
node2 NotReady  <none>    5m15s   v1.18.1
node3 NotReady  <none>    5m1s    v1.18.1

status node masih notReady karena belum deploy network add-on, dan node tidak bisa berkomunikasi satu sama lain.  
download calico network manifest dan ubah CIDR sesuaikan dan apply.

wget https://docs.projectcalico.org/v3.11/manifests/calico.yaml
vi calico.yaml #change 192.168.0.0/16 to 172.16.0.0/12
kubectl apply -f calico.yaml #wait a little
kubectl get node
NAME  STATUS    ROLES     AGE     VERSION
node1 Ready     master    15m42s   v1.18.1
node2 Ready     <none>    15m15s   v1.18.1
node3 Ready     <none>    15m1s    v1.18.1

untuk me-manage cluster dari host pc, tanpa login ke multipass/guest master-mode.

mkdir ~/.kube
multipass copy-files node1:.kube/config ~/.kube
kubectl get node
```

mematikan instance

```
multipass stop node10
multipass delete node10
multipass purge
```

### kembali ke default driver windows 10

```
multipass set local.driver=hyperv
```

```
code cloud_init.yaml::
users:
  - default
  - name: donald
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-rsa <onelinekey>
package_update: true
package_upgrade: true
packages:
  - nodejs
  - python3
multipass launch -c 2 -m 2G -d 20G -n donald-1-vm --cloud-init cloud-init.yaml
```
buka virtualbox, ubah port forward pada nat: host:2222 remote:22
konek ke multipass instance via ssh ubuntu@localhost -p2222

pada ubuntu host, enable ip forward dan iptables nat:
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo apt install ufw
sudo ufw limit ssh
ip a => check ip address
sudo iptables -t nat -I PREROUTING 1 -i eth0 -p tcp --dport 2222 -j DNAT --to-destination 10.0.2.15:22
sudo iptables -I FORWARD 1 -p tcp -d 10.0.2.15 --dport 22 -j ACCEPT
ssh ubuntu@10.0.2.15

### Access to host folder

```
multipass mount ~/share_folder donald-1-vm
multipass info donald-1-vm
multipass unmount donald-1-vm
```