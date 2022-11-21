sources:
- [https://pgillich.medium.com/setup-on-premise-kubernetes-with-kubeadm-metallb-traefik-and-vagrant-8a9d8d28951a](https://pgillich.medium.com/setup-on-premise-kubernetes-with-kubeadm-metallb-traefik-and-vagrant-8a9d8d28951a)

```
# Create Deployment
kubectl create deployment --image nginx:latest mynginx

# expose NodePort, simple tanpa ingress, random port range 30000-32767
kubectl expose deployment mynginx --name mynginx-nodeport --port=80 --type=NodePort
kubectl expose deployment mynginx --name mynginx-nodeport --port=80 --type=NodePort --dry-run=client -o yaml | kubectl patch -f - --type='json' --patch='[{"op":"add","path":"/spec/ports/0/nodePort","value":30800}]' --dry-run=client -o yaml | kubectl apply -f -
kubectl get node -o wide
kubectl get service mynginx-nodeport -o wide

# internal-ip & service port
curl -s 192.168.1.10:30800
curl -s 192.168.1.11:30800
curl -s 192.168.1.12:30800


# LoadBalancer: expose service on public ip from specified address pool. metalLB pilih ip paling bawah bebas dan membuat sebuah floating ip.
kubectl expose deployment mynginx --name mynginx-lb --port=80 --type=LoadBalancer --load-balancer-ip=192.168.1.253
kubectl get service mynginx-lb -o wide
curl -s 192.168.1.253:80

# Ingress : menyediakan beberapa service di IP:Port yg sama, dibedakan oleh HTTP routing rules (e.g hostname & path). external hostname `cluster-01.company.com`, ditambahkan di `/etc/hosts`
192.168.1.254 oam.cluster-01.company.com cluster-01.company.com



```
