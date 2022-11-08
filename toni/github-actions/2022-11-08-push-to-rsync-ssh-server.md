# Github Actions :: Deploy to SSH Server via RSYNC

Tujuan: menggunakan github actions untuk men-deploy zellwk.com. ketika kita push ke github,
github actions melakukan site build & deploy ke Digital Ocean droplet/server.

# 1. Generate SSH Key

```sh
cd ~/.ssh
ssh-keygen -t rsa -b 4096 -C "whypeout@nousers.mydomain.com" -f whypeout
ls
# pastikan ada file
whypeout.pub #public key
whypeout     #private key
```

# 2. Tambahkan Public key to `authorized_keys`

kita perlu menambahkan public key (whypeout.pub) ke server (authorized_keys) agar private key (whypeout)
bisa digunakan utk mengakses server.

```sh
cat whypeout.pub >> ~/.ssh/authorized_keys
# atau
cat whypeout.pub | ssh user@server ">> ~/.ssh/authorized_keys"
# atau
ssh-copy-id -i whypeout user@server
```

# 3. Tambahkan private key ke repository

1. Buka Repository
2. Klik Settings (repository, bukan account settings)
   ![settings](https://zellwk.com/images/2021/github-actions-deploy/github-secrets-location.png)
4. di Panel Kiri, klik Secret
   ![secret](https://zellwk.com/images/2021/github-actions-deploy/github-secrets-location.png)
6. klik New repository secret
   Name: SSH_PRIVATE_KEY
   Value: copy-paste isi whypeout (private key)
   ![new-secret](https://zellwk.com/images/2021/github-actions-deploy/new-repository-secret-button.png)
   ![add-new](https://zellwk.com/images/2021/github-actions-deploy/adding-a-secret.png)
   ![whypeout](https://zellwk.com/images/2021/github-actions-deploy/private-key.png)
   ![fill-here](https://zellwk.com/images/2021/github-actions-deploy/paste-secret-value.png)
   ![filled](https://zellwk.com/images/2021/github-actions-deploy/ssh-private-key.png)
7. klik Add secret

# 4. Tambahkan private key ke Github Actions Workflow

```yaml
steps:
  - name: Install SSH Key
    uses: shimataro/ssh-key-action@v2
    with:
      key: ${{ secrets.SSH_PRIVATE_KEY }}
  - name: Adding known_hosts
    run: ssh-keyscan -H ${{ secret.SSH_HOST }} >> ~/.ssh/known_hosts
```

Tambahkan secret,  
- SSH_HOST berisi IP Server kita  
- SSH_USER berisi username kita di server

# 5. Tambahkan value known_hosts

cek known_hosts server kita, dg perintah:

```sh
ssh-keyscan -H <server-ip-address>
```

# 6. Rsync ke Server

```sh
# base command
rsync -flags source user@server:destination
# example
rsync -avz ./dist/ ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:/home/zellwk/zellwk.com/dist/
```

# 7. Tambahkan secret Custom Port 

SSH_PORT == 2211

Tambahkan known_hosts dg custom port

```yaml
name: Deploy with rsync
run: ssh-keyscan -p ${{ secrets.SSH_PORT }} -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts
```

jika ada error, pastikan `-p` sebelum `-H`.

Tambahkan ke command rsync

```sh
rsync -avz -e "ssh -p ${{ secrets.SSH_PORT }}" ./dist/ ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:/home/zellwk/zellwk.com/dist/
```

# 8. Full Workflow `.github/workflow/deploy.yml`

```yaml
name: deploy
on:
  push:
    branches:
      - main
      - master
  schedule:
    - cron: '0 0 * * *' # Everyday at 12am
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1

      - run: npm install
      - run: npm run build

      - name: Install SSH Key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          known_hosts: unnecessary

      - name: Adding Known Hosts
        run: ssh-keyscan -p ${{ secrets.SSH_PORT}} -H ${{ secrets.SSH_HOST }}  >> ~/.ssh/known_hosts

      - name: Deploy with rsync
        run: rsync -avz -e "ssh -p ${{ secrets.SSH_PORT }}" ./dist/ ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:/var/www/zellwk.com/

      - name: Restart Node Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            cd /var/www/zellwk.com
            git fetch origin master
            git reset --hard FETCH_HEAD
            git clean -d -f --exclude secrets
            npm install --production
            chown $(whoami) . # PM2 doesn't recognize root user from Github Actions
            npm run restart
```

source:
- [github actions deploy via ssh](https://zellwk.com/blog/github-actions-deploy/)
- [github actions deploy rsync via ssh custom port](https://zellwk.com/blog/rsync-with-github-actions-when-using-a-custom-port/)





