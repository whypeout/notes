# Install Docker on Ubuntu



# Install Minikube on Ubuntu

```sh
apt update && apt upgrade -y
apt -y install qemu-kvm libvirt-daemon-system libvirt-daemon bridge-utils kubernetes-client

wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 -O minikube
wget https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2
chmod 755 minikube docker-machine-driver-kvm2
mv minikube docker-machine-driver-kvm2 /usr/local/bin/
minikube version

usermod -aG libvirt ubuntu
usermod -aG docker ubuntu

minikube start --driver=docker --memory=4096 
minikube status
minikube service list
minikube docker-env
minikube cluster-info
minikube get nodes
minikube ssh

hostname
docker ps
exit

minikube stop
minikube start
minikube delete
```

# Minikube Basic Usage

```sh
kubectl create deployment test-nginx --image=nginx
kubectl get pods
kubectl exec test-nginx-<id> -- env
kubectl exec -it test-nginx-<id> -- bash
root@test-nginx<id>:/# hostname && exit
kubectl logs test-nginx-<id>

```
