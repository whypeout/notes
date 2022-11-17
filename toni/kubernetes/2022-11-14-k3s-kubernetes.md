# Install k3s

```
curl -fsSL https://get.k3s.io | sh -s - --docker --write-kubeconfig-mode 644
curl -fsSL https://get.k3s.io | INSTSALL_K3S_SYMLINK=skip sh -s - --docker --write-kubeconfig-mode 644

curl -fsSL https://get.k3s.io | sudo sh -
k3s kubectl get node

# k3s symlink in /usr/local/bin/k3s kubectl crictl
# autocomplete kubectl
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source <(kubectl completion bash)

k3s check-config
kubectl cluster-info
kubectl get nodes -o wide

# wait all pods & deployment running & completed => ready & status column
kubectl get all -A -o wide
kubectl get endpoints -A
sudo k3s crictl ps -a
kubectl top pod --containers -A
docker ps -a
```

or create K3os VM

# Install helm

```
wget -O helm.tar.gz https://get.helm.sh/helm-v3.10.2-linux-amd64.tar.gz
tar xf helm.tar.gz
mv helm /usr/local/bin/helm
```

# Kubernetes Dashboard

- `kubectl proxy` => http://localhost:8001/api/v1/namespaces/<namespace>/services/<http_proto>:<service>:<port>/proxy/

# Install Traefik (built-in k3s)

```
sudo nano /var/lib/rancher/k3s/server/manifests/traefik.yaml
...
spec:
  chart: https://%{KUBERNETES_API}%/static/charts/traefik-1.81.0.tgz
  valuesContent: |-
    dashboard:
      enabled: true
      domain: "fqdn.local"
    rbac:
      enabled: true
...
#save&exit
watch kubectl get endpoints traefik-dashboard -n kube-system
```

> note: after reboot, helm file & ingress config recovered to original content

- Access Traefik Dashboard trough kubectl proxy

```
kubectl proxy
```

open [http://localhost:8001/api/v1/namespaces/kube-system/services/http:traefik-dashboard:80/proxy/dashboard/#/](http://localhost:8001/api/v1/namespaces/kube-system/services/http:traefik-dashboard:80/proxy/dashboard/#/)

- Access Traefik Dashboard trough Ingress

> ingress config doesn't support set path

open [http://local.dom/dashboard/](http://local.dom/dashboard/)

![img](https://miro.medium.com/max/720/1*QDvZW3UKBy_ULGE3Ezrj2A.png)

# Install Kubernetes Dashboard

```
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/alternative.yaml
```

Dashboard RBAC config:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

wait it ready

```
wait kubectl get endpoints kubernetes-dashboard -n kubernetes-dashboard
```

get bearer tokin for login:

```
kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token
```

- Access Kubernetes Dashboard trough kubectl proxy

```
kubectl proxy
```

open [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/)

- Access Kubernetes Dashboard trough ingress controller

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: "traefik"
    traefik.ingress.kubernetes.io/rule-type: "PathPrefixStrip"
spec:
  rules:
  - host: oam.internal
    http:
      paths:
      - path: /kubernetes
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 80
EOF
```

open [https://local.dom/kubernetes/](https://local.dom/kubernetes/)

![img](https://miro.medium.com/max/720/1*GncsTVu6aBdAJ6Swy6QdjA.png)

# Simple web server

```
# create server
kubectl create deployment --image nginx:latest my-nginx
kubectl expose deployment my-nginx --port=80

# check server
kubectl get all -o wide
# wait till deployment.apps/my-nginx READY 1/1
kubectl get pod/my-nginx-<id> -o yaml
kubectl get deployment.apps/my-nginx -o yaml
kubectl get service/my-nginx -o wide
# use cluster-ip & port to connect
curl http://10.43.234.108:80

# check service dns
kubectl run cluster-tester -it --rm --restart=Never image=busybox:1.28
#or kubectl run cluster-tester -it --rm --restart=Never --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
nslookup kubernetes.default
nslookup my-nginx.default.svc.cluster.local
wget -qS0- 

```


# Install Portainer

```sh
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
- [https://pgillich.medium.com/setup-lightweight-kubernetes-with-k3s-6a1c57d62217](https://pgillich.medium.com/setup-lightweight-kubernetes-with-k3s-6a1c57d62217)
