source: [download latest powershell 7.2 here](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2)

[install latest chocolatey here](https://chocolatey.org/install)

```
choco install docker-engine
choco install docker-cli
choco install docker-desktop
choco install nodejs
choco install openssh bitvise-ssh-client bitvise-ssh-server
choco install openssl
choco install kubernetes-node kubernetes-cli kubernetes-helm minikube kind rancher-desktop rancher-cli k3d k3sup rke civo-cli 
choco install git
```

[how to install kind kubernetes with choco in windows 10](https://blog.nillsf.com/index.php/2020/08/28/running-kind-in-windows/)

1.run powershell as administrator
2.copas :
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
3.run 
```
refreshenv
```
4.install docker-desktop & golang
```
choco install docker-desktop -y
choco install golang -y
```
5.restart system
```
shutdown -r
```
6.install kind (kubernetes)
```
choco install kind -y
```
7.create cluster
```
kind create cluster
kubectl get nodes
```
