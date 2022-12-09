# Docker & Kubernetes:: Multi-Node Local Kubernetes Cluster - KubeAdm - KinD (K8s in Docker)

source:
- [https://www.bogotobogo.com/DevOps/Docker/Docker-Kubernetes-Multi-Node-Local-Clusters-kind.php](https://www.bogotobogo.com/DevOps/Docker/Docker-Kubernetes-Multi-Node-Local-Clusters-kind.php)

Kubernetes in Docker (kind) [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/) adl tool utk menjalankan Kubernetes cluster di lokal menggunakan Docker container sbg "nodes". kita bisa membuat kubernetes cluster multi-node atau multi-control-plane.

## Install

- via go & docker: `kind` akan diinstall di `$(go env GOPATH/bin)`
```
export PATH=$PATH:$(go env GOPATH/bin)
export GOPATH=$(go env GOPATH)
go get -u sigs.k8s.io/kind
```

## Kind Basic Command

```
kind -h
  build
    kind build node-image
  completion
    kind completion bash > ~/.kind-completion && source ~/.kind-completion
    kind completion zsh > /usr/local/share/zsh/site-functions/_kind && autoload -U compinit && compinit
  create
    kind create cluster --config <file.yaml> --image <kindest.image> --kubeconfig <path-kubeconfig> --name <clustername> --wait 300
  delete
    kind delete cluster --kubeconfig ~/.kube/clustername-config --name <clustername>
    kind delete clusters
  export
    kind export kubeconfig --internal --kubeconfig ~/.kube/cluster-config --name <clustername>
    kind export logs <path> --name <clustername>
  get
    kind get clusters
    kind get kubeconfig --internal -n --name <clustername>
    kind get nodes -A --all-clusters -n --name <clustername>
  help
  load
    kind load docker-image <image> <images...> -n --name <clustername> --nodes <dest-nodes>
    kind load image-archive  <image.tar> -n --name <clustername> --nodes <dest-nodes>
  version
```

## Create a Cluster

```
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.13.4) 🖼 
 ✓ Preparing nodes 📦 
 ✓ Creating kubeadm config 📜 
 ✓ Starting control-plane 🕹️ 
Cluster creation complete. You can now use the cluster with:

$ export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
$ kubectl cluster-info
```
node image bisa dicari di docker hub [https://hub.docker.com/r/kindest/node/](https://hub.docker.com/r/kindest/node/).
setelah membuat cluster, kita bisa memakai `kubectl` utk berinteraksi via konfig file yg digenerate oleh `kind`:
```
$ export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"

$ echo $KUBECONFIG
~/.kube/kind-config-kind

$ cat $KUBECONFIG
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...
    server: https://localhost:64708
  name: kind
contexts:
- context:
    cluster: kind
    user: kubernetes-admin
  name: kubernetes-admin@kind
current-context: kubernetes-admin@kind
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0...
   client-key-data: LS0...
```
secara default cluster akan diberinama `kind`. kita bisa berikan custom nama utk cluster context name.
```
$ kubectl config get-contexts
CURRENT   NAME                    CLUSTER   AUTHINFO           NAMESPACE
*         kubernetes-admin@kind   kind      kubernetes-admin

$ kubectl cluster-info
Kubernetes master is running at https://localhost:56897
KubeDNS is running at https://localhost:56897/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
utk melihat semua cluster
```
$ kind get clusters
kind
```
melihat letak kubeconfig. jika kita punya beberapa cluster, akan ada beberapa config, tergantung nama cluster
```
$ kind get kubeconfig-path
~/.kube/kind-config-kind

$ kubectl get nodes -o wide
NAME                 STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kind-control-plane   Ready     master    23m       v1.13.4   172.17.0.2    <none>        Ubuntu 18.04.1 LTS   4.9.125-linuxkit   docker://18.6.3

$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23m

$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                  NAMES
88879275d8c2        kindest/node:v1.13.4   "/usr/local/bin/entr…"   About an hour ago   Up About an hour    56897/tcp, 127.0.0.1:56897->6443/tcp   kind-control-plane
```

## Create Nodes spesifik Config

```
# three node (two workers) cluster config
# apiVersion terupdate cek di: https://kind.sigs.k8s.io/docs/user/configuration/
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```
kita buat cluster dg `--config`
```
$ unset KUBECONFIG

$ kind delete cluster

$ export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"

$ kind create cluster --config kind_worker
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.13.4) 🖼
 ✓ Preparing nodes 📦📦📦 
 ✓ Creating kubeadm config 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Joining worker nodes 🚜 
Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
kubectl cluster-info

$ echo $KUBECONFIG
~/.kube/kind-config-kind

$ kubectl cluster-info
Kubernetes master is running at https://localhost:53350
KubeDNS is running at https://localhost:53350/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl config get-contexts
CURRENT   NAME                    CLUSTER   AUTHINFO           NAMESPACE
*         kubernetes-admin@kind   kind      kubernetes-admin 

$ kubectl get nodes
NAME                 STATUS    ROLES     AGE       VERSION
kind-control-plane   Ready     master    4m51s     v1.13.4
kind-worker          Ready     <none>    4m35s     v1.13.4
kind-worker2         Ready     <none>    4m34s     v1.13.4
```

## Deploy Flask

- Dockerfile
```
FROM alpine
RUN apk add --no-cache python3 && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools && \
    rm -r /root/.cache
COPY . /app
WORKDIR /app
RUN pip3 install -r requirements.txt
ENTRYPOINT [ "python3" ]
CMD [ "app.py" ]
```
- App.py
```
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def blog():
    return "Flask in kind Kubernetes cluster"

if __name__ == '__main__':
    app.run(threaded=True,host='0.0.0.0',port=8087)
```
- requirements.txt
```
Flask==0.10.1
```
Run COntainer di local, bukan di cluster:
```
$ docker build -t k8s-flask:latest .

$ docker images
REPOSITORY     TAG        IMAGE ID            CREATED             SIZE
k8s-flask      latest     51227513e1ef        2 minutes ago       60.6MB

$ docker run -p 5000:8087 k8s-flask:latest
5ca7b4861798fdc86886f5f86e43180028b44e9ec07186fadae21f94dc785a52

$ docker ps
CONTAINER ID  IMAGE             COMMAND           CREATED          STATUS          PORTS                   NAMES
5ca7b4861798  k8s-flask:latest  "python3 app.py"  12 seconds ago   Up 12 seconds   0.0.0.0:5000->8787/tcp   recursing_clarke
```
![img](https://www.bogotobogo.com/DevOps/Docker/images/Docker-Kind-Cluster/k8s-Flask.png)























