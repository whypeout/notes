how to install [minikube on ubuntu 20.04](https://mustafaakin.dev/posts/2020-04-11-installing-a-multi-node-kubernetes-cluster-with-ubuntu-multipass/)

## Why MiniKube ? 
minikube bisa menggunakan VirtualBox, KVM, Podman dan Docker.  
disini kita akan menggunakan Docker.  

Minikube adl tool utk mempermudah membuat cluster kubernetes di local.  
Minikube menjalankan single-node kubernetes cluster didalam VM di local machine.  

Minikube best fit karena:
- runs on windows, linux and macos
- support latest kubernetes release
- support multiple container runtime. containerd, kvm, podman etc
- support advance feature. load balancing, featuregates and filesystem mounts
- support addons. addons sbg marketplace antar developer share config utk running di minikube
- support ci environment

## Prerequire

- 2 cpu/more
- 2 gb ram/more
- 20gb disk free space
- good internet
- container/virtual machine manager. docker kvm podman virtualbox dll

# Install Minikube di Ubuntu 20.04

apt update && apt upgrade -y
curl -fsSL https://get.docker.com | sudo sh -
apt update && apt install docker-ce docker-compose docker-ce-cli
systemctl enable docker
systemctl start docker
systemctl status docker
useradd --create-home whypout
usermod -aG docker whypout

# Install KubeCtl on ubuntu 20.04

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(<kubectl.sha256 kubectl)" | sha256 --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
ls -la /usr/local/bin/kubectl
kubectl version --client

[https://www.downloadkubernetes.com/](https://www.downloadkubernetes.com/)

# Install Minikube on ubuntu 20.04

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
minikube version

[https://github.com/kubernetes/minikube](https://github.com/kubernetes/minikube)

# Start Minikube 

minikube start
kubectl cluster-info
kubectl config view

# Enabel Kubernetes Dashboard

minikube dashboard

[https://computingforgeeks.com/how-to-run-minikube-on-kvm/](https://computingforgeeks.com/how-to-run-minikube-on-kvm/)
https://mathiashueber.com/[pci-passthrough-ubuntu-2004-virtual-machine/](https://mathiashueber.com/pci-passthrough-ubuntu-2004-virtual-machine/)