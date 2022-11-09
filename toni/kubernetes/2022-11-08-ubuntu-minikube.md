# Install Docker on Ubuntu

```sh
apt update
apt upgrade
apt install ca-certificates curl gnupg lsb-release

mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# or manually download from
https://download.docker.com/linux/ubuntu/dists/<focal>/pool/stable/<amd64>/
# files
containerd.io_<version>_<arch>.deb
docker-ce_<version>_<arch>.deb
docker-ce-cli_<version>_<arch>.deb
docker-compose-plugin_<version>_<arch>.deb
# install it with
dpkg -i ./containerd.io.deb ./docker-ce.deb ./docker-ce-cli.deb ./docker-compose-plugin.deb

# or install with script
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# uninstall docker completely
apt-get purge docker-ce docker-ce-cli containerd.io docker-compose-plugin
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

# after installed
groupadd docker
usermod -aG docker ubuntu
newgrp docker
chown -R ubuntu:ubuntu /home/ubuntu/.docker/
chmod -R g+rwx /home/ubuntu/.docker/

# enable disable docker
systemctl enable docker.service containerd.service
systemctl disable docker.service containerd.service

# edit service
systemctl edit docker.service
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375

# or configure /etc/docker/daemon.json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}

# restart after edit
systemctl daemon-reload
systemctl restart docker

# cek service
ss -ntap|grep dockerd
netstat -tulpn|grep dockerd
```

Docker RootLess

```sh
$ curl -o rootless-install.sh -fsSL https://get.docker.com/rootless
$ sh rootless-install.sh
$ export PATH=$HOME/bin:$PATH

docker context use rootless
docker run hello-world

systemctl --user start|stop docker
nano /home/user/.config/systemd/user/docker.service

dockerd-rootless-setuptool.sh uninstall
~/bin/rootlesskit rm -rf ~/.local/share/docker ~/.config/docker

# remove under ~/bin
containerd containerd-shim containerd-shim-runc-v2 ctr docker docker-init docker-proxy dockerd dockerd-rootless-setuptool.sh dockerd-rootless.sh rootlesskit rootlesskit-docker-proxy runc vpnkit

# minikube rootless docker
dockerd-rootless-setuptool.sh install -f
docker context use rootless
minikube start --driver=docker --container-runtime=containerd
```


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

# standar docker
minikube start --driver=docker --memory=4096 
minikube config set driver docker

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

# KubeCtl

> kubectl harus ada di 1 versi minor dg cluster. misal kubectl v1.25 client bisa konek ke cluster v1.24, v1.25 dan v1.26. sebisa mungkin gunakan versi kubectl terbaru utk control plane tersebut.

```sh
# download latest kubectl with curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO https://dl.k8s.io/release/v1.25.0/bin/linux/amd64/kubectl
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# kubectl: OK

# install kubectl as root
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# install kubectl non-root user
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
# and then append (or prepend) ~/.local/bin to $PATH
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# check version
kubectl version --client
kubectl version --client --output=yaml    


# install via snap
snap install kubectl --classic
kubectl version --client

# verify kubectl config
# kubectl perlu kubeconfig utk bisa terkoneksi dg kubernetes cluster. kubeconfig di generate otomatis ketika membuat cluster oleh kube-up.sh atau minikube cluster. secara default, kubectl config berada di `~/.kube/connfig`.
kubectl cluster-info
kubectl cluster-info dump
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

# Minikube Multi Node

```sh
minikube start --nodes 3 -p multinode-demo --driver=docker
kubectl get nodes

minikube status -p multinode-demo
kubectl apply -f hello-deployment.yaml
kubectl rollout status deployment/hello
kubectl apply -f hello-svc.yaml
kubectl get pods -o wide

minikube service list -p multinode-demo
curl http://192.168.49.2:31000 #coba beberapa kali utk melihat perubahan response
```

- hello-deployment.yaml

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 100%
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      affinity:
        # memastikan pods akan berjalan pada hosts berbeda
        podAntiAffinity:
          requiredDuringSchedulingIgnoreDuringExecution:
          - labelSelector:
              matchExpressions: [{ key: app, operator: In, values: [hello] }]
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: hello-from
        image: pbitty/hello-from:latest
        ports:
          - name: http
            containerPort: 80
      terminationGracePeriodSeconds: 1
```

hello-svc.yaml

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  type: NodePort
  selector:
    app: hello
  ports:
    - protocol: TCP
      nodePort: 31000
      port: 80
      targetPort: http
```


source:
- [https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=2](https://www.server-world.info/en/note?os=Debian_11&p=minikube&f=2)
- [https://minikube.sigs.k8s.io/docs/drivers/docker/](https://minikube.sigs.k8s.io/docs/drivers/docker/)
- [https://rootlesscontaine.rs/getting-started/docker/](https://rootlesscontaine.rs/getting-started/docker/)
- [https://minikube.sigs.k8s.io/docs/tutorials/multi_node/](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)
- [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
