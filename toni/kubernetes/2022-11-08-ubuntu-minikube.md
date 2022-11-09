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
# create pods
kubectl create deployment test-nginx --image=nginx

# list all pods
kubectl get pods

# execute command inside pods
kubectl exec test-nginx-<id> -- env
kubectl exec -it test-nginx-<id> -- bash
root@test-nginx<id>:/# hostname && exit

# see pods logs
kubectl logs test-nginx-<id>

# scale up pods
kubectl scale deployment test-nginx --replicas=3
kubectl get pods

# set service
kubectl expose deployment test-nginx --type="NodePort" --port 80
kubectl get services test-nginx
minikube service test-nginx --url
curl http://192.168.39.79:30900

# delete service
kubectl delete services test-nginx

# delete pods
kubectl delete deployment test-nginx
```


source:
- [https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=2](https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=2)
- [https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=1](https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=1)
