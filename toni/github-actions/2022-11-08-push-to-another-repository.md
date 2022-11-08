----
title: "Github Actions :: Push to Another Repository"
date: 2022-11-08 09:39
tags: ["github", "actions", "repository"]
category: ["github"]
----

# Setup Github SSH Deploy Keys

## Generate SSH Keys

```
ssh-keygen -t ed25519 -C "github_deploy_keys@gmail.com"
```

ssh-keygen akan meminta path utk menyimpan key, dg format yg mudah diingat
misal: `/home/$USER/.ssh/<id-github>-<repo-name>`,
jadi `/home/$USER/.ssh/whypeout-notes`,
usahakan untuk membuat key untuk masing-masing repository.

> biarkan `passphrase` nya kosong, karena github action tidak support.

## Tambahkan Public Keys ke Destination Repository

1. Buka Repository tujuan (Public), misal : [https://github.com/cpina/push-to-another-repository-output](https://github.com/cpina/push-to-another-repository-output)
2. Klik "Settings" (untuk repository, bukan account settings)
   ![repo-settings](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-10.png)
3. Di Panel Kiri bagian bawah, Klik "Deploy Keys"
   ![deploy keys](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-20.png)
4. Tambahkan deploy key "add deploy key"
   ![add key](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-30.png)
   - Title: "GitHub Action push to another repository"
   - Key: paste isi public key, buka file `/home/$USER/.ssh/whypeout-notes.pub`
   - Enable / ceklis "Allow write access"
   ![add-new](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-40.png)

## Tambah Private Key ke Source Repository

1. Buka repository Source/Sumber (Private), misal : [https://github.com/cpina/push-to-another-repository-deploy-keys-example](https://github.com/cpina/push-to-another-repository-deploy-keys-example)
2. Klik "Settings" (setting pada repository, bukan account settings)
   ![src-settings](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-10.png)
3. Di Panel Kiri, klik `Secrets` Lalu klik `Actions`
   ![secret-action](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-50.png)
4. Klik `New repository secret`
   ![new-repo-secret](https://cpina.github.io/push-to-another-repository-docs/_images/ssh-key-60.png)
   - Name: "SSH_DEPLOY_KEY"
   - Value: paste isi private key, buka `/home/$USER/.ssh/whypeout-notes`
   ```
   -------BEGIN OPENSSH PRIVATE KEY-------
   ...
   -------END OPENSSH PRIVATE KEY-------
   ```
# Complete `.github/workflow/ci.yml`

```yaml
# This is a basic workflow to help you get started with Actions

name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: pandoc/latex
    steps:
      - uses: actions/checkout@v2
      - name: Install mustache (to update the date)
        run:  apk add ruby && gem install mustache
      - name: creates output
        run:  sh ./build.sh
      - name: debugging (files are kept between steps)
        run: pwd; ls -l
      - name: Pushes to another repository
        id: push_directory
        uses: cpina/github-action-push-to-another-repository@main
        env:
          API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
        with:
          source-directory: output/
          destination-github-username: 'cpina'
          destination-repository-name: 'push-to-another-repository-output'
          user-email: carles3@pina.cat
          commit-message: See ORIGIN_COMMIT from $GITHUB_REF
          target-branch: main
      - name: Test get variable exported by push-to-another-repository
        run: echo $DESTINATION_CLONED_DIRECTORY
```


SOURCE:  
- [cpina - push to another repository](https://cpina.github.io/push-to-another-repository-docs/setup-using-ssh-deploy-keys.html)
