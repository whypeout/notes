sources:
- [https://blog.nootch.net/post/kubernetes-at-home-with-k3s/](https://blog.nootch.net/post/kubernetes-at-home-with-k3s/)

# Basic Kubernetes

- **Pods** basic work unit, bisa berupa container tunggal, atau beberapa container. Pods akan selalu ada disetiap node dalam cluster. k8s menyediakan IP utk pods. containers didalam pod bisa saling terkoneksi, tapi containers tidak bisa terkoneksi antar pods. kita tidak boleh mengakses pods secara langsung. itu tugas Services
- **Services** untuk mempermudah memanage sbg sebuah unit yg berisi beberapa pods. ReplicaSets memanage Pods secara langsung, bukan Services. Services mengenali Pods yg dikontrol dg `labels`
- **Labels** setiap object di k8s memiliki metadata. label salah satunya, dan annotaions. actions dlm k8s bs di scoped dlm selector dg menyediakan spesifik label dari target
- **Volumes** spt docker, menghubungkan containers ke storage. production umumnya menggunakan S3/sejenisnya. utk homelab bisa dg `hostPath` (atau NFS Server) mount type yg map langsung ke folder di server. k8s memperumit dg membuat `PersistentVolume` (PV) dan `PersistentVolumeClaim` (PVC) utk mengaksesnya. kita pakai `hostPath` direct mount di Deployment, tapi PV & PVC punya control lebih thd volumes.
- **Deployment** main work units. declare Docker image to use, service deployment via labels, volumes to mount, ports to export & optional security concern.
- **ConfigMaps** store config data in key-value form. env utk deployment disimpan di ConfigMap, semua atau spesifik keys.
- **Ingress** tanpa ini, pods bisa running, tapi tidak ter-exposed keluar. `nginx` ingress slh satunya.
- **Jobs** & **CronJobs** workload sekali/berulang-kali yg dpt dijalankan.

1. Install Linux on VM Virtualbox

gunakan fitur Clone VM daripada snapshot

![clone](https://blog.nootch.net/img/post/k8s-at-home/vbox-clone.png)

pastikan network nya "Bridged" dg mac-addreess sama dg master (vm asal clone) agar ip nya tetap. atau gunakan ip statis didalam vm nya

![network](https://blog.nootch.net/img/post/k8s-at-home/vbox-mac.png)

pastikan port forward http 80/tcp dan https 443/tcp

forward juga port ssh 22/tcp, k8s api 6443/tcp dan kubelet 10250/udp,
jika tidak berada di network yg sama (remote digitalocean, azure, amazon dll)

pastikan public key dari ssh key yg akan kita gunakan sudah terinstall di server

```
# nano ~/.ssh/config
Host k3s
  User root
  Hostname 192.168.200.244
  IdentityFile ~/.ssh/id_rsa
# ssh-copy-id -i ~/.ssh/id_rsa root@192.168.200.244
```

2. Install kubectl

```
export HOST_IP=192.168.200.244
export HOST=k3s
KUBECTL=kubectl --kubectl ~/.kube/k3s-vm-config

ssh $HOST 'export INSTALL_K3S_EXEC=" --no-deploy servicelb --no-deploy traefik"; \
 curl -fsSL https://get.k3s.io | sh -'
scp $HOST:/etc/rancher/k3s/k3s.yaml .
sed -r 's/(\b[0-9]{1,3}\.){3}[0-9]{1,3}\b'/"$HOST_IP"/ k3s.yaml > ~/.kube/k3s-vm-config && rm k3s.yaml
```

- `servicelb` kita tidak perlu loadbalancing pada single server
- `traefik` kita akan menggunakan nginx sbg ingresses, tidak perlu install ingress controller

baris kedua, copy `k3s.yaml` dari server (created saat instalasi dan termasuk cert utk konek ke server)
baris ketiga, me-replace ip local server `127.0.0.1` ke `192.168.200.244` ip remote server, karena kita akses dari luar server
kubectl mengambil config dari ~/.kube

2.5 Clients (optional)

[K9s](https://github.com/derailed/k9s) CLI

![k9s](https://blog.nootch.net/img/post/k8s-at-home/k9s.png)

```
go install github.com/derailed/k9s@latest

curl -fsSL https://webinstall.dev/k9s | bash

docker run --rm -it -v $KUBECONFIG:/root/.kube/config quay.io/derailed/k9s

k9s help
k9s info
k9s -n portainer
k9s --context minikube
k9s --readonly
```

[Lens](https://k8slens.dev/) GUI 

- [https://api.k8slens.dev/binaries/Lens-2022.11.101953-latest.x86_64.AppImage](https://api.k8slens.dev/binaries/Lens-2022.11.101953-latest.x86_64.AppImage)  
- [https://api.k8slens.dev/binaries/Lens-2022.11.101953-latest.amd64.deb](https://api.k8slens.dev/binaries/Lens-2022.11.101953-latest.amd64.deb)  

![lens](https://blog.nootch.net/img/post/k8s-at-home/lens.png)

pastikan config/context di folder `~/.kube` sudah benar

3. Nginx Ingress, Letsencrypt dan storage

```
kubectl apply -f k8s/ingress-nginx-v1.04.yml
kubectl wait --namespace ingress-nginx \
 --for=condition=ready pod \
 --selector=app.kubernetes.io/component=controller \
 --timeout=60s
kubectl apply -f k8s/cert-manager-v1.0.4.yaml
kubectl wait --namespace=cert-manager \
 --for=condition=ready pod \
 --all --timeout=60s
kubectl apply -f k8s/lets-encrypt-staging.yml
kubectl apply -f k8s/lets-encrypt-prod.yml
```

- [cert-manager-v1.0.4.yml](https://gist.github.com/sardaukar/6a950ea9abcfe651e68876dbc6001fb0/raw/023b7b60a30a26c95deab154d4fc8a59812ffc85/cert-manager-v1.0.4.yaml)
- [ingress-nginx-v.1.0.4.yml](https://gist.github.com/sardaukar/6a950ea9abcfe651e68876dbc6001fb0/raw/023b7b60a30a26c95deab154d4fc8a59812ffc85/ingress-nginx-v1.0.4.yaml)
- [lets-encrypt-prod.yml](https://gist.github.com/sardaukar/6a950ea9abcfe651e68876dbc6001fb0/raw/023b7b60a30a26c95deab154d4fc8a59812ffc85/lets-encrypt-prod.yml)
- [lets-encrypt-staging.yml](https://gist.github.com/sardaukar/6a950ea9abcfe651e68876dbc6001fb0/raw/023b7b60a30a26c95deab154d4fc8a59812ffc85/lets-encrypt-staging.yml)

edit nginx-ingress line 323:

```
dnsPolicy: ClusterFirstWithHostNet
hostNetwork: true
```

4. Portainer

```yaml
# define portainer namespace
---
apiVersion: v1
kind: Namespace
metadata:
  name: portainer
# serviceAccount
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portainer-sa-clusteradmin
  namespace: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.0"
# portainer volume & claim
---
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    type: local
  name: portainer-pv
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/zpool/volumes/portainer/claim"
---
# Source: portainer/templates/pvc.yaml
kind: "PersistentVolumeClaim"
apiVersion: "v1"
metadata:
  name: portainer
  namespace: portainer
  annotations:
    volume.alpha.kubernetes.io/storage-class: "generic"
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.0"
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "1Gi"
---
# Source: portainer/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.0"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  namespace: portainer
  name: portainer-sa-clusteradmin
  
---
# Source: portainer/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.0"
spec:
  type: NodePort
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
      name: http
      nodePort: 30777
    - port: 9443
      targetPort: 9443
      protocol: TCP
      name: https
      nodePort: 30779
    - port: 30776
      targetPort: 30776
      protocol: TCP
      name: edge
      nodePort: 30776
  selector:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
---
# Source: portainer/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portainer
  namespace: portainer
  labels:
    io.portainer.kubernetes.application.stack: portainer
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
    app.kubernetes.io/version: "ce-latest-ee-2.16.0"
spec:
  replicas: 1
  strategy:
    type: "Recreate"
  selector:
    matchLabels:
      app.kubernetes.io/name: portainer
      app.kubernetes.io/instance: portainer
  template:
    metadata:
      labels:
        app.kubernetes.io/name: portainer
        app.kubernetes.io/instance: portainer
    spec:
      nodeSelector:
        {}
      serviceAccountName: portainer-sa-clusteradmin
      volumes:
        - name: portainer-pv
          persistentVolumeClaim:
            claimName: portainer
      containers:
        - name: portainer
          image: "portainer/portainer-ce:2.16"
          imagePullPolicy: Always
          args:
          - '--tunnel-port=30776'
          volumeMounts:
            - name: portainer-pv
              mountPath: /data
          ports:
            - name: http
              containerPort: 9000
              protocol: TCP
            - name: https
              containerPort: 9443
              protocol: TCP
            - name: tcp-edge
              containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          readinessProbe:
            httpGet:
              path: /
              port: 9443
              scheme: HTTPS
          resources:
            {}
# networking part
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: portainer-ingress
  namespace: portainer
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
spec:
  rules:
    - host: portainer.domain.tld
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: portainer
                port:
                  name: http
  tls:
    - hosts:
      - portainer.domain.tld
      secretName: portainer-staging-secret-tls
```

```
kubectl apply -k stacks/portainer
```

```yaml
# stacks/portainer/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - portainer.yaml
```

Kustomize : yaml generator template-free for kubectl

```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

docker pull k8s.gcr.io/kustomize/kustomize:v3.8.7
docker run k8s.gcr.io/kustomize/kustomize:v3.8.7 version

choco install kustomize

```

[Kompose](https://kompose.io/) : convert docker-compose.yml to k8s yaml

```
kompose convert -f docker-compose.yaml
kubectl apply -f .
kubectl get pods

curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose-linux-amd64 -o kompose

https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose-windows-amd64.exe
choco install kubernetes-kompose

go get -u github.com/kubernetes/kompose

https://github.com/kubernetes/kompose/releases

wget https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose_1.26.1_amd64.deb # Replace 1.26.1 with latest tag
sudo apt install ./kompose_1.26.1_amd64.deb

docker build -t kompose https://github.com/kubernetes/kompose.git
docker run --rm -it -v $PWD:/opt kompose sh -c "cd /opt && kompose convert"
```


