# Acme Install in Linux

source:
- [https://www.techrepublic.com/article/how-to-create-lets-encrypt-ssl-certificates-with-acme-sh-on-linux/](https://www.techrepublic.com/article/how-to-create-lets-encrypt-ssl-certificates-with-acme-sh-on-linux/)
- [https://docs.ukfast.co.uk/domains/ssl/letsencrypt/letsencrypt_acme_sh.html](https://docs.ukfast.co.uk/domains/ssl/letsencrypt/letsencrypt_acme_sh.html)


```
# prereq
sudo -i
apt update -y
apt upgrade -y
apt install curl wget socat -y

# install with curl/wget & bash
curl https://get.acme.sh | sh
wget -O- https://get.acme.sh | sh

# update path env
source ~/.bashrc

# verify version
acme.sh --version

# update/upgrade to latest
acme.sh --upgrade --auto-upgrade

## Issue Cert
acme.sh --issue -d fathon.id -d *.fathon.id --webroot /var/www/fathon.id

acme.sh --issue -d example.com -d www.example.com -w /path/to/doc/root --reloadcmd "systemctl reload <webserver>"

acme.sh --issue -d test.example.com -w /path/to/doc/root --reloadcmd "systemctl reload <webserver>"

acme.sh --issue -d fathon.id -d www.fathon.id --webroot /var/www/fathon.id --keylength 2048|4096|8192|ec-256|ec-384

acme.sh --issue -d fathon.id --standalone

## Copy to web server directory
acme.sh --install-cert -d fathon.id --cert-file /etc/ssl/certs/fathon.pem --key-file /etc/ssl/keyfile/fathon.pem --fullchain-file /etc/ssl/certs/fullchain/fullchain.pem --reloadcmd "sudo systemctl reload apache2.service"

acme.sh --issue -d example.com -d www.example.com --nginx --reloadcmd "systemctl reload nginx"

acme.sh --issue -d test.example.com --nginx --reloadcmd "systemctl reload nginx"

acme.sh --issue -d test.example.com --nginx --reloadcmd "systemctl reload nginx"
acme.sh --issue -d example.com -d www.example.com --apache --reloadcmd "systemctl reload apache2"



## Renew cert
acme.sh --renew -d fathon.id --force

```