# Install k3s

```
curl -fsSL https://get.k3s.io | sudo sh -
k3s kubectl get node
```

or create K3os VM

# Install helm

```
wget -O helm.tar.gz https://get.helm.sh/helm-v3.10.2-linux-amd64.tar.gz
tar xf helm.tar.gz
mv helm /usr/local/bin/helm
```

# Install Portainer

```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm repo add portainer https://portainer.github.io/k8s/
helm repo update

helm install --create-namespace -n portainer portainer portainer/portainer

export NODE_PORT=$(kubectl get --namespace portainer -o jsonpath="{.spec.ports[1].nodePort}" services portainer)
export NODE_IP=$(kubectl get nodes --namespace portainer -o jsonpath="{.items[0].status.addresses[0].address}")
echo https://$NODE_IP:$NODE_PORT
```

or 

# Install k3s with Docker (ZFS Support)

> note: pls use fresh install ubuntu/debian os

```
curl -fsSL https://get.docker.com/ | CHANNEL=stable sh
systemctl enable --now docker

# or use rancher installation script
curl https://releases.rancher.com/install-docker/20.10.sh | sh

curl -fsSL https://get.k3s.io | sh -s - --docker

kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml

# add flags --disable=traefik to disable traefik on k3s installation
```

```
sudo k3s kubectl get pods --all-namespaces
sudo docker ps
```

# Install K3s Manually

1. download k3s from [https://github.com/k3s-io/k3s/releases/latest](https://github.com/k3s-io/k3s/releases/latest)
2. run the server
```
sudo k3s server &
# kubeconfig /etc/rancher/k3s/k3s.yaml
sudo k3s kubectl get nodes

# from remote node. export NODE_TOKEN=$(cat /var/lib/rancher/k3s/server/node-token)
sudo k3s agent --server https://your-remote-server:6443 --token ${NODE_TOKEN}
```

# K3d = K3s inside Docker

- via docker RUN

```
sudo docker run \
  -d --tmpfs /run \
  --tmpfs /var/run \
  -e K3S_URL=${SERVER_URL} \
  -e K3S_TOKEN=${NODE_TOKEN} \
  --privileged rancher/k3s:vX.Y.Z
```

- via [docker-compose.yml](https://github.com/k3s-io/k3s/blob/master/docker-compose.yml)

```yaml
# to run define K3S_TOKEN, K3S_VERSION is optional, eg:
#   K3S_TOKEN=${RANDOM}${RANDOM}${RANDOM} docker-compose up

version: '3'
services:

  server:
    image: "rancher/k3s:${K3S_VERSION:-latest}"
    command: server
    tmpfs:
    - /run
    - /var/run
    ulimits:
      nproc: 65535
      nofile:
        soft: 65535
        hard: 65535
    privileged: true
    restart: always
    environment:
    - K3S_TOKEN=${K3S_TOKEN:?err}
    - K3S_KUBECONFIG_OUTPUT=/output/kubeconfig.yaml
    - K3S_KUBECONFIG_MODE=666
    volumes:
    - k3s-server:/var/lib/rancher/k3s
    # This is just so that we get the kubeconfig file out
    - .:/output
    ports:
    - 6443:6443  # Kubernetes API Server
    - 80:80      # Ingress controller port 80
    - 443:443    # Ingress controller port 443

  agent:
    image: "rancher/k3s:${K3S_VERSION:-latest}"
    tmpfs:
    - /run
    - /var/run
    ulimits:
      nproc: 65535
      nofile:
        soft: 65535
        hard: 65535
    privileged: true
    restart: always
    environment:
    - K3S_URL=https://server:6443
    - K3S_TOKEN=${K3S_TOKEN:?err}

volumes:
  k3s-server: {}
```

```
docker-compose up --scale agent=3
kubectl --kubeconfig kubeconfig.yaml get node
```

source:
- [https://forum.openmediavault.org/index.php?thread/40524-how-to-install-k3s-a-simple-kubernetes-to-do-almost-better-than-truenas-scale-wi/](https://forum.openmediavault.org/index.php?thread/40524-how-to-install-k3s-a-simple-kubernetes-to-do-almost-better-than-truenas-scale-wi/)
- [https://docs.k3s.io/advanced](https://docs.k3s.io/advanced)
- [https://github.com/k3s-io/k3s](https://github.com/k3s-io/k3s)
