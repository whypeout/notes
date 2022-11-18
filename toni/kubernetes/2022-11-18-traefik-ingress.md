# Mengarahkan trafik Kubernetes dg Traefik

source:
- [https://opensource.com/article/20/3/kubernetes-traefik](https://opensource.com/article/20/3/kubernetes-traefik)
- [source code all config](https://gitlab.com/carpie/ingressing_with_k3s/-/archive/master/ingressing_with_k3s-master.zip)

## Deploy Konfig

buat sebuah file `mysite.yaml` berisi:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysite-nginx
  labels:
    app: mysite-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysite-nginx
  template:
    metadata:
      labels:
        app: mysite-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

berikut kita deklarasikan dg nama deployment: mysite-nginx, label app: mysite-nginx, dengan replica:1 pod.
kita juga buat hanya 1 container dg nama nginx. gunakan docker image nginx dari dockerhub oleh k3s.
terakhir containerPort 80 yg berarti "didalam container" pod akan listen di port 80.  

didalam container, penting. port hanya akan listen didalam container, hanya ada di internal network.
ini penting, karena kita akan membuat beberapa container listen di container port yg sama.
dg kata lain, dg konfig ini, beberapa pod dapat listen di container port 80 tanpa terjadi conflict.
untuk mengijinkan access ke pod, kita perlu konfigurasi "service".

## Service Konfig

dalam kubernetes, service adl abstraksi. mengijinkan akses ke 1 atau beberapa pods.
saat konek ke service & service merutekan ke 1 pod atau load balance ke multiple pods jika menggunakan multiple pods replica.

kita bisa satukan service dalam satu file konfigurasi, kita pisahkan bagian konfigurasi dg `---`, kita tambahkan ke `mysite.yaml`

```
---
apiVersion: v1
kind: Service
metadata:
  name: mysite-nginx-service
spec:
  selector:
    app: mysite-nginx
  ports:
    - protocol: TCP
      port: 80
```

disini kita deklarasikan, nama service mysite-nginx-service. kita pakai *selector* `app:mysite-nginx`.
ini bagaimana service memilih container app mana utk di route. disni kita pakai `app` label utk container kita `mysite-nginx`.
service akan menggunakan app label utk menemukan countainer kita. kita tentukan protocol service nya TCP dan service listen nya port 80.

## Ingress Konfig

ingress konfig menentukan bagaimana trafik dari luar kluster ke service didalam kluster.
k3s sudah pre-config dg Traefik sbg ingress controller. selanjutnya kita tambahkan ke `mysite.yaml`:

```
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: mysite-nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: mysite-nginx-service
          servicePort: 80
```

deklarasi, nama ingress record nya `mysite-nginx-ingress`, kemudian kita expect `traefik` sbg ingress controller dg annotations `kubernetes.io/ingress.class`.
di bagian `rules`, ketika trafik `http` datang dan `path` nya `/` (atau dibawahnya), akan dirutekan ke service `backend` dg nama spesifik `serviceName mysite-nginx-service` dan merutekan ke `servicePort 80`.
disni trafik masuk HTTP dikonekkn ke service yg kita buat sebelumnya.

## Deploy Something

karena kita ingin men-deploy sesuatu, selain halaman default nginx. kita deploy saja `index.html`

```html
<html>
<head><title>K3S!</title>
  <style>
    html {
      font-size: 62.5%;
    }
    body {
      font-family: sans-serif;
      background-color: midnightblue;
      color: white;
      display: flex;
      flex-direction: column;
      justify-content: center;
      height: 100vh;
    }
    div {
      text-align: center;
      font-size: 8rem;
      text-shadow: 3px 3px 4px dimgrey;
    }
  </style>
</head>
<body>
  <div>Hello from K3S!</div>
</body>
</html>
```

karena kita belum sampe di mekanisme storage di kubernetes, kita pake trik kotor dg menyimpan file ini di configMap kubernetes.
ini tidak direkomendasikan utk production.

```
kubectl create configMap mysite-html --from-file index.html
```

buat resource `configMap` dg nama `mysite-html` dari local file `index.html`.
file(s) ini akan di simpan (store) didalam kubernetes resource yg dpt kita call nanti di konfigurasi.
umumnya digunakan utk menyimpan config file (sesuai namanya), selanjutnya kita diskusikan solusi storage yg baik di kubernetes.

dg configmap, kita mount didalam container `nginx`. kita lakukan 2 step.
pertama kita buat volume dg configmap. lalu kita mount volume kedalam container nginx.
kita tambahkan dibawah `spec` label setelah `containers` di `mysite.yaml`.

```
      volumes:
      - name: html-volume
        configMap:
          name: mysite-html
```

kita info kubernetes utk memakai `volume` dg nama `html-volume` dan volume harus berisi konten `configMap` dg nama `html-volume` yg kita buat sebelumnya.

lalu di nginx container spec(ification), dibawah `ports`, kita tambah:

```
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
```

kita info kubernetes, utk container `nginx`, kita mau mount `volume` dg nama `html-volume` dg mount-path (didalam container) `/usr/share/nginx/html` (default HTML dir).
dg mount volume pada path tsb, kita replace default konten dg isi volume.

bagian config `deployment` menjadi :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysite-nginx
  labels:
    app: mysite-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysite-nginx
  template:
    metadata:
      labels:
        app: mysite-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: mysite-html
```

sekarang kita siap deploy

```
kubectl apply -f mysite.yaml

# output:
deployment.apps/mysite-nginx created
service/mysite-nginx-service created
ingress.networking.k8s.io/mysite-nginx-ingress created

# periksa pods
kubectl get pods
```

status: ContainerCreating => container masih dalam proses, blm running, tunggu beberapa saat sblm cek lagi dg `kubectl get pods`
karena pertama kali k3s blm download nginx image utk membuat pod.

## Try It

![https://opensource.com/sites/default/files/uploads/mysite.jpg](https://opensource.com/sites/default/files/uploads/mysite.jpg)

## Another One

sekarang kita sudah punya k3s cluster menjalankan sebuah website. 
kita buat website lain utk dideploy. kita tambahkan folder ke configmap.

```
kubectl create configmap mydog-html --from-file html
```

kita copy `mysite.yaml` ke `mydog.yaml`, edit dan sesuaikan

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydog-nginx
  labels:
    app: mydog-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mydog-nginx
  template:
    metadata:
      labels:
        app: mydog-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: mydog-html
---
apiVersion: v1
kind: Service
metadata:
  name: mydog-nginx-service
spec:
  selector:
    app: mydog-nginx
  ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: mydog-nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
    traefik.frontend.rule.type: PathPrefixStrip
spec:
  rules:
  - http:
      paths:
      - path: /mydog
        backend:
          serviceName: mydog-nginx-service
          servicePort: 80
```

sederhananya, search&replace `mysite` ke `mydog`. edit lain dibagian ingress. kita ubah `path` ke `/mydog` dan kita tambahkan annotation, `traefik.frontend.rule.type:PathPrefixStrip`.

spec path `/mydog` memberitahu traefik utk route semua incoming req dg req path dimulai `/mydog` ke `mydog-nginx-service`. path lain akan lanjut di route ke `mysite-nginx-service`.

annotation baru `PathPrefixStrip`, memberitahu Traefik utk strip off prefix `/mydog` sebelum kirim req ke `mydog-nginx-service`.
untungnya, mydog-nginx app tidak expect prefix. 












