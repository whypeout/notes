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

5. Samba share

menjalankan samba server in-cluster.

```yaml
# stacks/samba/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

secretGenerator:
  - name: smbcredentials
    envs:
    - auth.env

resources:
  - deployment.yaml
  - service.yaml
---
# stacks/samba/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: smb-server
spec:
  ports:
    - port: 445
      protocol: TCP
      name: smb
  selector:
    app: smb-server
---
# stacks/samba/deployment.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: smb-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: smb-server
  strategy:
    type: Recreate
  template:
    metadata:
      name: smb-server
      labels:
        app: smb-server
    spec:
      volumes:
        - name: smb-volume
          hostPath:
            path: /zpool/shares/smb
            type: DirectoryOrCreate
      containers:
        - name: smb-server
          image: dperson/samba
          args: [
            "-u",
            "$(USERNAME1);$(PASSWORD1)",
            "-u",
            "$(USERNAME2);$(PASSWORD2)",
            "-s",
            # name;path;browsable;read-only;guest-allowed;users;admins;writelist;comment
            "share;/smbshare/;yes;no;no;all;$(USERNAME1);;mainshare",
            "-p"
          ]
          env:
            - name: PERMISSIONS
              value: "0777"
            - name: USERNAME1
              valueFrom:
                secretKeyRef:
                  name: smbcredentials
                  key: username1
            - name: PASSWORD1
              valueFrom:
                secretKeyRef:
                  name: smbcredentials
                  key: password1
            - name: USERNAME2
              valueFrom:
                secretKeyRef:
                  name: smbcredentials
                  key: username2
            - name: PASSWORD2
              valueFrom:
                secretKeyRef:
                  name: smbcredentials
                  key: password2
          volumeMounts:
            - mountPath: /smbshare
              name: smb-volume
          ports:
            - containerPort: 445
              hostPort: 445
```

kita tidak pakai Persistent Volume dan Claim, hanya direct `hostPath`. dg `type: DirectoryOrCreate` utk membuat directory jika blm ada.

kita pakai docker image dari [dperson/samba](https://github.com/dperson/samba) yg bisa setting share dg mudah. kita buat single share, dg 2 user (username1 sbg admin share). username & password dari file berikut.

auth file:

```
# stacks/samba/auth.env
username1=alice
password1=foo

username2=bob
password2=bar
```

jalankan stack ke cluster

```
kubectl apply -k stacks/samba
```

6. BookStack

contoh convert docker-compose.yml menggunakan Kompose ke K8s, kita pakai BookStack, sebuah wiki app.

```
#original docker-compose.yml
version: '2'
services:
  mysql:
    image: mysql:5.7.33
    environment:
    - MYSQL_ROOT_PASSWORD=secret
    - MYSQL_DATABASE=bookstack
    - MYSQL_USER=bookstack
    - MYSQL_PASSWORD=secret
    volumes:
    - mysql-data:/var/lib/mysql
    ports:
    - 3306:3306

  bookstack:
    image: solidnerd/bookstack:21.05.2
    depends_on:
    - mysql
    environment:
    - DB_HOST=mysql:3306
    - DB_DATABASE=bookstack
    - DB_USERNAME=bookstack
    - DB_PASSWORD=secret
    volumes:
    - uploads:/var/www/bookstack/public/uploads
    - storage-uploads:/var/www/bookstack/storage/uploads
    ports:
    - "8080:8080"

volumes:
 mysql-data:
 uploads:
 storage-uploads:
```

convert dg Kompose:

```
> kompose convert -f bookstack-original-compose.yaml
WARN Unsupported root level volumes key - ignoring
WARN Unsupported depends_on key - ignoring
INFO Kubernetes file "bookstack-service.yaml" created
INFO Kubernetes file "mysql-service.yaml" created
INFO Kubernetes file "bookstack-deployment.yaml" created
INFO Kubernetes file "uploads-persistentvolumeclaim.yaml" created
INFO Kubernetes file "storage-uploads-persistentvolumeclaim.yaml" created
INFO Kubernetes file "mysql-deployment.yaml" created
INFO Kubernetes file "mysql-data-persistentvolumeclaim.yaml" created
```

kompose tidak support `depends_on` dan `volumes`. kita akan fix nanti.

```yaml
# stacks/bookstack/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - bookstack-build.yaml
```

```yaml
# stacks/bookstack/bookstack-build.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    io.kompose.service: bookstack
  name: bookstack
spec:
  ports:
  - name: bookstack-port
    port: 10000
    targetPort: 8080
  - name: bookstack-db-port
    port: 10001
    targetPort: 3306
  selector:
    io.kompose.service: bookstack
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: bookstack-storage-uploads-pv
spec:
  capacity:
    storage: 5Gi
  hostPath:
    path: >-
      /zpool/volumes/bookstack/storage-uploads
    type: DirectoryOrCreate
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-path
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    io.kompose.service: bookstack-storage-uploads-pvc
  name: bookstack-storage-uploads-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path
  volumeName: bookstack-storage-uploads-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: bookstack-uploads-pv
spec:
  capacity:
    storage: 5Gi
  hostPath:
    path: >-
      /zpool/volumes/bookstack/uploads
    type: DirectoryOrCreate
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-path
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    io.kompose.service: bookstack-uploads-pvc
  name: bookstack-uploads-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path
  volumeName: bookstack-uploads-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: bookstack-mysql-data-pv
spec:
  capacity:
    storage: 5Gi
  hostPath:
    path: >-
      /zpool/volumes/bookstack/mysql-data
    type: DirectoryOrCreate
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-path
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    io.kompose.service: bookstack-mysql-data-pvc
  name: bookstack-mysql-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path
  volumeName: bookstack-mysql-data-pv
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: bookstack-config
  namespace: default
data:
  DB_DATABASE: bookstack
  DB_HOST: bookstack:10001
  DB_PASSWORD: secret
  DB_USERNAME: bookstack
  APP_URL: https://bookstack.domain.tld
  MAIL_DRIVER: smtp
  MAIL_ENCRYPTION: SSL
  MAIL_FROM: user@domain.tld
  MAIL_HOST: smtp.domain.tld
  MAIL_PASSWORD: vewyvewysecretpassword
  MAIL_PORT: "465"
  MAIL_USERNAME: user@domain.tld
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: bookstack-mysql-config
  namespace: default
data:
  MYSQL_DATABASE: bookstack
  MYSQL_PASSWORD: secret
  MYSQL_ROOT_PASSWORD: secret
  MYSQL_USER: bookstack
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    io.kompose.service: bookstack
  name: bookstack
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: bookstack
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        io.kompose.service: bookstack
    spec:
      containers:
      - name: bookstack
        image: reddexx/bookstack:21112
        securityContext:
          allowPrivilegeEscalation: false
        envFrom:
        - configMapRef:
            name: bookstack-config
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: bookstack-uploads-pv
          mountPath: /var/www/bookstack/public/uploads
        - name: bookstack-storage-uploads-pv
          mountPath: /var/www/bookstack/storage/uploads
      - name: mysql
        image: mysql:5.7.33
        envFrom:
        - configMapRef:
            name: bookstack-mysql-config
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: bookstack-mysql-data-pv
      volumes:
      - name: bookstack-uploads-pv
        persistentVolumeClaim:
          claimName: bookstack-uploads-pvc
      - name: bookstack-storage-uploads-pv
        persistentVolumeClaim:
          claimName: bookstack-storage-uploads-pvc
      - name: bookstack-mysql-data-pv
        persistentVolumeClaim:
          claimName: bookstack-mysql-data-pvc
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookstack-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
spec:
  rules:
    - host: bookstack.domain.tld
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: bookstack
                port:
                  name: bookstack-port
  tls:
    - hosts:
      - bookstack.domain.tld
      secretName: bookstack-staging-secret-tls
```


