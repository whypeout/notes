# Cheat Sheet

## KubeCTL

sources:
- [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)


- Autocomplete

```
# bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# or
alias k=kubectl
complete -o default -F __start_kubectl k

# zsh
source <(kubectl completion zsh)
echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc
```

- Shorthand utk `--all-namespaces` gunakan `-A`. contoh: `kubectl -A`

- kubectl context & config

```
# melihat kubeconfig yg sedang digunakan (termasuk merged)
kubectl config view

# menggunakan beberapa kubeconfig bersamaan (merged)
KUBECONFIG=~/.kube/config:~/.kube/kubconfig2
kubectl config view

# melihat password user e2e 
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'

kubectl config view -o jsonpath='{.users[].name}'   # username pertama
kubectl config view -o jsonpath='{.users[*].name}'  # list semua user
kubectl config get-contexts                         # list semua context
kubectl config current-context                      # context saat ini
kubectl config use-context <myclustername>          # berpindah context
kubectl config set-context <myclustername>          # berinama cluster di kubeconfig

# gunakan proxy utk mengakses server ini
kubectl config set-cluster <myclustername> --proxy-url=my-proxy-url

# tambah user baru ke kubeconfig dg basic auth
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# simpan namespace scr permanent utk semua cmd kubectl di context ini
kubectl config set-context --current --namespace=ggckad-s2

# set context utk menggunakan spesifik username & namespace
kubectl config set-context gce --user=cluster-admin --namespace=foo \
  && kubectl config use-context gce

# delete user foo
kubectl config unset users.foo

# alias utk set/show context/namespace (bash&compatible, set current context sebelum menggunakan kn utk set namespace)
alias kx='f() { [ "$1" ] && kubectl config use-context $1 || kubectl config current-context ; } ; f'
alias kn='f() { [ "$1" ] && kubectl config set-context --current --namespace $1 || kubectl config view --minify | grep namespace | cut -d" " -f6 ; } ; f'
```

- kubectl apply

`apply` resource apps lewat file define (manifest yaml). melakukan update resource didalam cluster (recommended on production)

- membuat objects

kubernetes manifests bisa di define lewat YAML atau JSON. file extentions `.yaml`, `.yml` dan `.json`

```
kubectl apply -f ./manifest.yaml    # create resource(s)
kubectl apply -f ./file1.yaml -f file2.yaml # dari beberapa file
kubectl apply -f ./dir              # buat dari file manifest didalam folder
kubectl apply -f https://git.repo.com/user/manifest.yaml # buat dari url
kubectl create deployment nginx --image=nginx   # buat instance tunggal nginx

# buat job utk print hello-world
kubectl create job hello --image=busybox:1.28 -- echo "Hello World"

# buat cronjob print hello world setiap menit
kubectl create cronjob hello --image=busybox:1.28 --schedule="*/1 * * * *" -- echo "Hello World"

# documentation utk pod manifest
kubectl explain pods

# buat multiple yaml object lewat stdin
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000"
EOF

# buat secret dg beberapa keys
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF
```

- lihat dan cari resources

```





























