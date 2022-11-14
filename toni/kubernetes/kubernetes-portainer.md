# Portainer : Menambahkan Kubernetes Env

source:
- [https://thenewstack.io/portainer-how-to-add-a-kubernetes-environment/](https://thenewstack.io/portainer-how-to-add-a-kubernetes-environment/)

Portainer dapat dikoneksikan ke remote kubernetes cluster pada LAN yg sama dan FQDN.
Bicara tentang Portainer, salah satu platform management utk multiple env,
dimana kita bisa pisahkan sesuai keperluan. misal satu lokal env utk tim development,
satu azure env utk tim deployment.

edge agent, salah satu env yg dapat ditambahkan utk kubernetes, yg memungkinkan utk memanage:
- namespaces
- helm
- applications
- configMaps & secrets
- cluster

ketika kita create app di portainer kubernetes env, kita bisa gunakan manifest atau
menggunakan portainer GUI (mirip dg menambahkan fullstack/container deployment pada docker env)

portainer merupakan salah satu GUI berbasis web ter-solid utk memanage kubernetes env

persiapan sebelum memanage kubernetes env pada portainer, kita install docker, disini kita pakai ubuntu 20.04 server

```
apt update
apt install ca-certificates curl gnupg lsb-release wget apt-transport-https -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee -a /etc/apt/sources.list.d/docker.list
apt update
apt install docker-ce docker-ce-cli containerd.io -y
systemctl enable --now docker
usermod -aG docker ubuntu
newgrp docker
```

selanjutnya kita install minikube dan kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo cp minikube-linux-amd64 /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube
```

kemudian kita deploy portainer server instance sbg docker container

```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 --name=portianer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.16
```

sekarang kita buat cluster kubernetes dan menambahkannya ke portainer server instance

![image](https://cdn.thenewstack.io/media/2022/06/330efb40-portkube1.jpg)
![image](https://cdn.thenewstack.io/media/2022/06/299271e1-portkube2.jpg)

disini kita tambahkan kubernetes via nodeport, lalu run command di kubernetes host.

```
curl -L https://downloads.portainer.io/portainer-agent-ce211-k8s-nodeport.yaml -o portainer-agent-k8s.yaml; kubectl apply -f portainer-agent-k8s.yaml
```

lanjutkan dg mengisi form pada portainer add new env, pada kubernetes server address, isikan: `https://192.168.1.13:9001`

![image](https://cdn.thenewstack.io/media/2022/06/64d1f1bb-portkube3.jpg)

masukkan env ke group, jika perlu. pada dropdown jika sudah pernah buat  
pilih tags, jika diperlukan/sudah pernah buat  
klik add env. setelah semua proses selesai, kembali ke main window/home utk melihat daftar env, kubernetes env

![image](https://cdn.thenewstack.io/media/2022/06/f11ff537-portkube4.jpg)





