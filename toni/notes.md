# Side Notes

> List ini belum di test & dokumentasikan

## DevOps

- Ansible & Terraform: AWS [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Terraform-null_resource-local-exec-remote-exec-triggers.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Terraform-null_resource-local-exec-remote-exec-triggers.php)
- Ansible Modules [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Modules.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Modules.php)
- Ansible Playbooks [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Playbooks.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Playbooks.php)
- Ansible Handlers [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Handlers.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Handlers.php)
- Ansible Roles [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Roles.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Roles.php)
- Ansible Deploy GoApp to MiniKube [https://www.bogotobogo.com/DevOps/Ansible/Ansible-Deploying-a-Go-App-to-Minikube.php](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Deploying-a-Go-App-to-Minikube.php)

## Kubernetes
- Kind utk Kubernetes test PR : [https://mauilion.dev/posts/kind-k8s-testing/](https://mauilion.dev/posts/kind-k8s-testing/)

## Docker 
- Serba Serbi [https://gulaliit.medium.com/install-command-docker-application-77ae1b85af39](https://gulaliit.medium.com/install-command-docker-application-77ae1b85af39)

## TMUX
- Kostumize TMUX [https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)
- Setup Terminal dg ZSH TMUX [https://dev.to/andrenbrandao/terminal-setup-with-zsh-tmux-dracula-theme-48lm](https://dev.to/andrenbrandao/terminal-setup-with-zsh-tmux-dracula-theme-48lm)
- Navigate TMUX [https://dev.to/waylonwalker/how-i-navigate-tmux-in-2021-2ina](https://dev.to/waylonwalker/how-i-navigate-tmux-in-2021-2ina)
- 

## MikroTik

- CHR Resize Disk [https://forum.mikrotik.com/viewtopic.php?t=124515](https://forum.mikrotik.com/viewtopic.php?t=124515)
```
# stop vm
# increase image size
qemu-img resize chr1.img +30G
# start vm
```
- CHR Container [https://help.mikrotik.com/docs/display/ROS/Container](https://help.mikrotik.com/docs/display/ROS/Container)
- CHR REST API (+ Reverse Proxy) [https://help.mikrotik.com/docs/display/ROS/REST+API](https://help.mikrotik.com/docs/display/ROS/REST+API)
- CHR RouterOS [https://help.mikrotik.com/docs/pages/viewpage.action?pageId=18350234](https://help.mikrotik.com/docs/pages/viewpage.action?pageId=18350234)
- IoT MQTT Telemetry [https://help.mikrotik.com/docs/display/UM/MQTT+and+ThingsBoard+configuration](https://help.mikrotik.com/docs/display/UM/MQTT+and+ThingsBoard+configuration)
- CHR SNMP [https://help.mikrotik.com/docs/display/ROS/SNMP](https://help.mikrotik.com/docs/display/ROS/SNMP)
- Custom Rate-Limit (Upto) [https://www.o-om.com/2020/06/optimasi-custom-rate-limit-yang-sesuai.html](https://www.o-om.com/2020/06/optimasi-custom-rate-limit-yang-sesuai.html)
- Queue HTB Parent-Child [https://simpony.web.id/2020/11/fakta-menarik-pada-queue-winbox-mikrotik.html](https://simpony.web.id/2020/11/fakta-menarik-pada-queue-winbox-mikrotik.html)
- REST API with AXIOS Javascript [https://github.com/renomureza/ros-rest](https://github.com/renomureza/ros-rest)
- DNS over HTTPS cloudflare [https://jcutrer.com/howto/networking/mikrotik/mikrotik-dns-over-https](https://jcutrer.com/howto/networking/mikrotik/mikrotik-dns-over-https)
```
/ip dns static
add name=cloudflare-dns.com address=104.16.248.249
add name=cloudflare-dns.com address=104.16.249.249
add name=dns.google address=8.8.8.8
add name=dns.google address=8.8.4.4
/ip dns set servers=8.8.8.8,8.8.4.4
/ip dns set use-doh-server=https://cloudflare-dns.com/dns-query verify-doh-cert=yes
/logging add topics=dns,debug
/tool torch interface=wan enable=src-addr,dst-addr,port,protocol filter=port=443
```
- RouterOS Recovery Password from Backup File [https://jcutrer.com/howto/networking/mikrotik/recover-routeros-passwords-from-backup](https://jcutrer.com/howto/networking/mikrotik/recover-routeros-passwords-from-backup)
```
python3 -V
mkdir rostest && cd rostest
git clone https://github.com/BigNerd95/RouterOS-Backup-Tools.git

virtualenv ./env
source ./venv/Scripts/activate
pip install pycryptodome

python RouterOS-Backup-Tools/ROSbackup.py unpack myrouter.backup -d myrouter
python RouterOS-Backup-Tools/extract_user.py myrouter/user.dat
```
