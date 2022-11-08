## Prerequisites
- [GitHub](https://github.com) Repository
- Pengetahuan Dasar [Github Actions](https://docs.github.com/en/actions/learn-github-actions)
- Workflow YML file di `.github/workflow/`

## Pengenalan

untuk membuat workflow seperti yg diinginkan, mungkin perlu beberapa percobaan.
berikut beberapa ide yg mungkin dapat membantu:
- [Dump semua Github Actions context](https://til.simonwillison.net/github-actions/dump-context)
- [Github Actions Debug Logging](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)
- Repository Kecil khusus experimen, dg pipeline yg bisa dijalankan dg cepat

## Github Events

workflow secara umum menggunakan trigger `on: [push]` utk setiap push pada semua branch.
berikut tambahan/fitur yg sering digunakan:

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

> default branch name is `main`  
> kita juga bisa menggunakan pull_request dg tipe yg spesifik.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, closed]
```

bagaimanapun git command tidak berjalan jika branch tersebut sudah dihapus setelah pull request merge.

solusi?
- membedakan antara event push & pull request:

```yaml
if: github.event_name == 'push'
```

- membedakan repo asal dari fork:

```yaml
if: github.event_name.pull_request.head.repo.full_name == github.repository
```

- mendapatkan head ref dari feature branch:

```yaml
github.event.pull_request.head.ref
```

- ambil informasi lain tentang pull request, misal title:

```yaml
github.event.pull_request.title
```

hal yg harus diperhatikan:
- `actions/checkout@v3` berperilaku berbeda
- `github.event.pull_request` properti akan kosong ketika di trigger oleh **push di main branch**
- `github.event.commits` properti akan kosong jika di trigger oleh **pull request event** 

> bacaan lanjutan: [event yg men-trigger workflow](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows)
> [lebih dalam tentang 'pull_request'](https://frontside.com/blog/2020-05-26-github-actions-pull_request)

## GIT Commands

menggunakan [git action untuk melakukan push ke origin](https://stackoverflow.com/questions/57921401/push-to-origin-from-github-action)
contoh melakukan push dg commit message dan author menggunakan env var:

```yaml
- name: GIT commit and push all changed files
  env:
    CI_COMMIT_MESSAGE: Continues Integration Build Artifacts
    CI_COMMIT_AUTHOR: Continues Integration
  run: |
    git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
    git config --global user.email "${{ username@users.noreply.github.com"
    git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
    git push
```

berikut contoh untuk commit hanya perubahan di folder docs:

```yaml
- name: GIT commit and push docs
  env: 
    CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
    CI_COMMIT_AUTHOR: Continuous Integration
  run: |
    git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
    git config --global user.email "username@users.noreply.github.com"
    git add docs
    git commit -m "${{ env.CI_COMMIT_MESSAGE }}"
    git push
```

> silahkan ganti email pada `username@users.noreply.github.com`

## All-in-one Solutions

tersedia beberapa solusi all-in-one sbg poin awal utk beberapa use case.
sbg gantinya, parameter dan perilaku mungkin berubah seiring waktu dan tidak se-stabil git commands.
agar lebih fleksibel dan kontrol, contoh dibawah mungkin lebih cocok. jika kesimpelan yg dicari.

- [git-auto-commit](https://github.com/stefanzweifel/git-auto-commit-action)
  solusi mudah utk: github action utk commit file dg 80% use case.
- [add-and-commit action](https://github.com/EndBug/add-and-commit)
  kontrol lebih: action mengijinkan kita memilih path yg akan di 'add' dan 'commit' perubahan
  sebagaimana kita gunakan git di pc.
  
### Pros

- tidak perlu git command
- bisa di ubah-atur
- dokumentasi yg baik

### Cons

- dependency tambahan
- hanya utk Unix/Linux systems
- [Example 1](#example1) sbg contoh

## Solusi Termudah

jika file yg di update & generate otomatis harus di push ke main branch repository, misal setelah pull request di merge,
maka berikut contoh yg bisa digunakan. berikut contoh workflow node.js, silahkan di adapt ke bahasa lain.

### Example1

```yaml
name: Continuous Integration
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    env: 
      CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
      CI_COMMIT_AUTHOR: Continuous Integration
    steps:
    - uses: actions/checkout@v3

    # Build steps
    - uses: actions/setup-node@v3
      with:
        node-version: '12' 
    - name: Node Install
      run: npm ci
    - name: Node Build (lint, test, coverage, doc, build, package)
      run: npm run package

    # Commit and push all changed files.
    - name: GIT Commit Build Artifacts (coverage, dist, devdist, docs)
      # Only run on main branch push (e.g. after pull request merge).
      if: github.event_name == 'push'
      run: |
        git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
        git config --global user.email "username@users.noreply.github.com"
        git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
        git push
```

> Note: tidak perlu mencegah workflow utk men-trigger workflow,
> karena [trigger workflow dari workflow](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows#triggering-a-workflow-from-a-workflow),
> menjelaskan bahwa "events triggered by `GITHUB_TOKEN` will not create a new workflow run".

> Note: jika kita menggunakan Personal Access Token untuk melakukan [actions/checkout](https://github.com/actions/checkout),
> workflow akan mentrigger dirinya sendiri, menghasilkan endless-loop. contoh berikut memperbaikinya.

> perhatian. ini tidak akan bekerja jika main branch di protected. buat Personal Access Token sbg Administrator
> dan menggunakannya sbg secret token sbb.

### Example2

variasi dari [example1](#example1) menggunakan Personal Access Token utk checkout utk memastikan pipeline tidak
dijalankan kembali secara otomatis setelah commit/push dibuat.

```yaml
name: Continuous Integration
on:
  push:
    branches:
      - main
    # Ignore changes in folders that are affected by the auto commit. (Node.js project)
    paths-ignore: 
      - 'coverage/**'
      - 'devdist/**'
      - 'dist/**'
      - 'docs/**'
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    env: 
      CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
      CI_COMMIT_AUTHOR: Continuous Integration
    steps:
    - uses: actions/checkout@v3
      with:
        token: ${{ secrets.WORKFLOW_GIT_ACCESS_TOKEN }}

    # Build steps
    - uses: actions/setup-node@v3
      with:
        node-version: '12' 
    - name: Node Install
      run: npm ci
    - name: Node Build (lint, test, coverage, doc, build, package)
      run: npm run package

    # Commit and push all changed files. 
    # Must only affect files that are listed in "paths-ignore".
    - name: GIT Commit Build Artifacts (coverage, dist, devdist, docs)
      # Only run on main branch push (e.g. pull request merge).
      if: github.event_name == 'push'
      run: |
        git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
        git config --global user.email "username@users.noreply.github.com"
        git add coverage devdist dist docs
        git commit -m "${{ env.CI_COMMIT_MESSAGE }}"
        git push
```

> folders di `paths-ignore` dan `git-add` harus sama utk mencegah workflow ter-trigger oleh dirinya sendiri.

## Jalankan steps pada auto-commit

jika beberapa steps pada workflow harus di eksekusi utk melakukan menggenerate commit/push otomatis,
commit name & author dapat di-compare utk men-deteksi auto commit run. setidaknya step yg membuat auto commit
harus di skip ketika auto commit men-trigger workflow. selanjutnya, Personal Access Token harus dibuat,
atau seluruh workflow akan di skip utk auto-created commit.

### auto-commit env var

```yaml
- name: Set environment variable "is-auto-commit"
  if: github.event.commits[0].message == env.CI_COMMIT_MESSAGE && github.event.commits[0].author.name == env.CI_COMMIT_AUTHOR
  run: echo "is-auto-commit=true" >> $GITHUB_ENV
```

> ini hanya berlaku utk event 'push'. var ini akan empty ketika 'pull_request'. "is-auto-commit" akan selalu "false".

Tambahkan display steps utk debugging dan maintenance sesuaikan:

```yaml
- name: Display Github event variable "github.event.commits[0].message"
  run: echo "last commit message = ${{ github.event.commits[0].message }}" 
- name: Display Github event variable "github.event.commits[0].author.name"
  run: echo "last commit author = ${{ github.event.commits[0].author.name }}" 
- name: Display environment variable "is-auto-commit"
  run: echo "is-auto-commit=${{ env.is-auto-commit }}"
```

### Skip push on auto-commit

env var diatas dbg digunakan utk skip step tertentu. setidaknya step utk push file baru ke repo harus di skip.

```yaml
- name: Commit build artifacts (dist, devdist, docs, coverage)
  # Only run on main branch push (e.g. pull request merge). 
  # Don't run again on an already pushed auto commit.
  if: github.event_name == 'push' && env.is-auto-commit == false
  run: |
    git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
    git config --global user.email "username@users.noreply.github.com"
    git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
    git push
```

## Example3

wajib menggunakan Personal Access Token

```yaml
name: Continuous Integration
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    env: 
      CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
      CI_COMMIT_AUTHOR: ${{ github.event.repository.name }} Continuous Integration
    steps:
    - uses: actions/checkout@v3
      with:
        token: ${{ secrets.WORKFLOW_GIT_ACCESS_TOKEN }}

    # Set environment variable "is-auto-commit" 
    - name: Set environment variable "is-auto-commit"
      if: github.event.commits[0].message == env.CI_COMMIT_MESSAGE && github.event.commits[0].author.name == env.CI_COMMIT_AUTHOR
      run: echo "is-auto-commit=true" >> $GITHUB_ENV

    # Display variables for debugging
    - name: Display Github event variable "github.event.commits[0].message"
      run: echo "last commit message = ${{ github.event.commits[0].message }}" 
    - name: Display Github event variable "github.event.commits[0].author.name"
      run: echo "last commit author = ${{ github.event.commits[0].author.name }}" 
    - name: Display environment variable "is-auto-commit"
      run: echo "is-auto-commit=${{ env.is-auto-commit }}"

    # Build (will also run on auto commit)
    - uses: actions/setup-node@v3
      with:
        node-version: '12' 
    - name: Install node packages
      run: npm ci
    - name: Build package (lint, test, build, package, merge)
      run: npm run package

    # Commit and push all changed files.
    - name: Display event name 
      run: echo "github.event_name=${{ github.event_name }}"
    - name: Commit build artifacts (dist, devdist, docs, coverage)
      # Don't run again on already pushed auto commit. Don't run on pull request events.
      if: env.is-auto-commit == false && github.event_name != 'pull_request'
      run: |
        git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
        git config --global user.email "joht@users.noreply.github.com"
        git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
        git push
```

...

source: [lebih dalam dengan github actions, by joht.github.io](https://joht.github.io/johtizen/build/2022/01/20/github-actions-push-into-repository.html#example-1)














