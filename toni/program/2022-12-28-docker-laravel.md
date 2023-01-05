## Laravel Docker
sources:
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04)
- [https://budasuyasa.medium.com/deploy-laravel-dengan-docker-ke-production-bcbed8738e5c](https://budasuyasa.medium.com/deploy-laravel-dengan-docker-ke-production-bcbed8738e5c)

## 1. download demo app

```sh
cd ~ && \
curl -L https://github.com/do-community/travellist-laravel-demo/archive/tutorial-1.0.1.zip -o travellist.zip
sudo apt update && \
sudo apt install unzip && \
unzip travellist.zip && \
move travellist-laravel-demo* travellist-demo && \
cd travellist-demo && \
cp .env.example .env && \
nano .env
```

## 2. dotenv

```dotenv
APP_NAME=Travellist 
APP_ENV=dev
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=travellist
DB_USERNAME=travellist_user
DB_PASSWORD=password
```

## 3. demo app dockerfile

kita perlu user baru utk menjalankan `artisan` dan `composer`.
`uid` membuat user didalam container sama dg uid sistem kita, agar permission didalam host sama dg didalam container.

```dockerfile
# base image
FROM php:7.4-fpm

# args dari docker-compose
ARG user
ARG uid

# update & install pkg
RUN apt update && apt install -y git curl \
    libpng-dev libonig-dev libxml2-dev zip unzip

# clean pkg
RUN apt clean && rm -rf /var/lib/apt/lists/*

# install & enable ext
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# install composer dari image composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# add new user to www-data and root
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user 
WORKDIR /var/www
USER $user 
```

## 4. setup nginx config & database dump files

buat konfig utk nginx

```
mkdir -p docker-compose/nginx
nano docker-compose/nginx/travellist.conf
```

```
server {
    listen 80;
    index index.php index.html;
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```

buat file sql yg akan diimport ketika membuat container mysql db
```
mkdir docker-compose/mysql
nano docker-compose/mysql/init_db.sql
```

```sql
DROP TABLE IF EXISTS `places`;

CREATE TABLE `places` (
    `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
    `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
    `visited` tinyint(1) NOT NULL DEFAULT '0',
    PRIMARY KEY (id)
) ENGINE=innoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

INSERT INTO places (name,visited) VALUES ('Berlin',0), ('Budapest',0), ('Cincinnati',1), ('Denver',0), ('Helsinki',0), ('Lisbon',0), ('Moscow',1), ('Nairobi',0), ('Oslo',1), ('Rio',0), ('Tokyo',0);
```

## 5.multi container env with docker-compose

```yaml
# nano docker-compose.yml
version: "3.7"

networks:
  travellist:
    driver: bridge

services:
  app:
    build:
      args:
        user: ubuntu
        uid: 1000
      context: ./
      dockerfile: Dockerfile
    image: travellist
    container_name: travellist-app
    restart: unless-stopped
    working_dir: /var/www/
    volumes:
      - ./:/var/www
    networks:
      - travellist

  db:
    image: mysql:5.7
    container_name: travellist-db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_USER: ${DB_USERNAME}
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - ./docker-compose/mysql:/docker-entrypoint-initdb.d
    networks:
      - travellist

  nginx:
    image: nginx:1.17-alpine
    container_name: travellist-nginx
    restart: unless-stopped
    ports:
      - 8080:80
    volumes:
      - ./:/var/www
      - ./docker-compose/nginx:/etc/nginx/conf.d
    networks:
      - travellist
```
keterangan: services `app`
- `build` perintahkan docker-compose utk melakukan build Dockerfile pada path (context) menjadi local image. inject args `user` dan `uid` kedalam Dockerfile saat build image
- `image` nama image yg akan dibuild
- `container_name` nama container utk service ini
- `restart`, selalu restart, kecuali service di stop
- `working_dir` default working dir utk service di `/var/www`
- `volumes` buat volume bind-mount local dir `./` ke `/var/www` didalam container
- `networks` buat service menggunakan network `travellist`

secara default, docker-compose menggunakan `.env` pada dir yg sama dg `docker-compose.yml`.
services `db` database mysql
- `image` pull dari docker hub
- `environment` ambil dari file `.env`
- `volumes` bind-mount dir tempat dump `.sql` yg akan digunakan utk meng-inisiasi database. image mysql akan mengimport `sql` file didalam folder `/docker-entrypoint-initdb.d/`

services `nginx`
- `image` pull dari docker hub
- `ports` akses external `8000` ke port internal docker `80`
- `volumes` sinkron `./` ke `/var/www` didalam container. dan config nginx di `/etc/nginx/conf.d/travellist.conf`

## After

```
# build image
docker-compose build app

# jalankan semua service / stack
docker-compose up -d

# melihat service yg berjalan
docker-compose ps

# run cmd didalam container, list file di workdir /var/www
docker-compose exec app ls -l

# install composer
docker-compose exec app composer install

# generate uniq app key
docker-compose exec app php artisan key:generate

# curl http://domain-ip:8000

# melihat log services
docker-compose logs nginx

# pause dan unpause
docker-compose pause
docker-compose unpause

# menghapus semua container services
docker-compose down
```

---

## Contoh Studi Kasus

- aplikasi di dev dg laravel
- app menggunakan server database dg host berbeda/terpisah. lebih aman dan skalable. tidak disarankan menjalankan database dalam container karena resiko kehilangan data dan sulit dibackup.
- app menggunakan redis sbg cache driver
- app menggunakan cloud storage utk menyimpan file (protokol aws s3). lebih aman dan skalable. opsi lain menggunakan local storage lewat volume
- app menjalankan scheduler dan worker

![image](https://user-images.githubusercontent.com/89820226/210689118-99af3699-fcdd-487f-af0f-d4bc6bd7f0ca.png)

1. di host kita install nginx sbg reverse proxy meneruskan request dari host ke container `web_server`
2. container yg dipakai `web_server`, `cache` dan `app`. dikoneksikan lewat `docker network` dijalankan lewat `docker-compose`
3. container `web_server` menjalankan app laravel dg service nginx dg root directory `public` laravel.
4. container `app` berisi seluruh source code, serivce php-fpm, worker dan scheduler. lewat docker network, service php-fpm terkoneksi dg service nginx didalam container `web_server`. proses php-fpm, workder dan scheduler disupervisi oleh supervisord.
5. container `cache` berisi service redis utk cache app. terkoneksi ke  container `app` lewat docker network
6. container `app` terhubung dg database server dan cloud storage yg berada diluar docker host.

### WorkFlow

1. docker repository
2. source code
3. build image diperlukan
4. tag dan push image ke container registry (docker hub / amazon container registry)
5. pull image ke production server
6. docker-compose.yml production
7. dotenv .env utk production
8. reverse proxy di host dg nginx

![image](https://user-images.githubusercontent.com/89820226/210696809-1f37b206-2e7d-4809-9680-c03a3824be0d.png)

#### 1. Menyiapkan Container Repository

Kita perlu siapkan container repository untuk menyimpan docker image yg akan kita gunakan. seperti Docker Hub atau AWS Elastic Container Registry.

kita buat akun dan login di [Docker Hub](hub.docker.com)

Buat repository, klik `Create Repository`

![image](https://user-images.githubusercontent.com/89820226/210701484-d6cb11f3-b951-4971-95d5-534843519dda.png)

image `app` akan kita push ke repository `laravel-prod-demo-app`. image `web_server` kita push ke `laravel-prod-demo-web_server`.

![image](https://user-images.githubusercontent.com/89820226/210701643-05531f22-4a46-4e1f-83c7-17e57665b269.png)
![image](https://user-images.githubusercontent.com/89820226/210701662-aea908f6-6586-4298-92f7-62d5c8dd6d02.png)

#### 2. Menyiapkan Source Code

pastikan sudah tidak ada error pada app. jika menggunakan laravel Mix, pastikan sudah mengcompile asset dg `npm run production` atau `npm run prod`.

contoh source code ambil disini : [https://github.com/budasuyasa/laravel-docker-prod](https://github.com/budasuyasa/laravel-docker-prod)

#### 3. Build Image

kita buat folder `Docker` di root directory source code laravel dan letakkan semua file yg kita perlukan didalamnya.

```
mkdir -p Docker/Dockerfile
cd Docker/Dockerfile
touch app.Dockerfile
```
```dockerfile
# base image
FROM php:7.4-fpm

# pindah workdir
WORKDIR /var/www/

# install deps, clean dan install php extension
RUN apt update -y && apt install -y build-essential libpng-dev \
    libjpeg62-turbo-dev libfreetype6-dev locales zip libonig-dev \
    libzip-dev jpegoptim optipng gifsicle ca-certificates vim \
    unzip tmux git cron supervisor curl    
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-jpeg=/usr/include/ --with-freetype=/usr/include/
RUN docker-php-ext-install gd

# install redis extension
RUN pecl install -o -f redis && rm -rf /tmp/pear && docker-php-ext-enable redis

# install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# copy file kedalam image, ubah ownership
COPY . /var/www/
COPY --chown=www-data:www-data . /var/www/
RUN chown -R www-data:www-data /var/www
RUN chown -R www-data:www-data /var/log/supervisor

# install composer libs deps
RUN composer install

# expose fpm
EXPOSE 9000

# copy dan jalankan service supervisord
COPY Docker/supervisor/ /etc/
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]

# ganti user aktif ke www-data
USER www-data
```

dalam app container, kita gunakan **supervisor** untuk memanage proses php-fpm, scheduler dan worker. karena dalam container hanya bisa menjalankan satu service, kita pakai supervisor untuk menjalankan lebih dari satu service didalam container.

referensi supervisord [https://supervisord.org/](https://supervisord.org/)

![image](https://user-images.githubusercontent.com/89820226/210717467-f8e0fd5d-0a50-489f-97b2-40c8df61ce59.png)

selanjutnya kita buat folder `supervisord` yg berisi konfig supervisor untuk memanage proses didalam container nantinya.

```
mkdir Docker/supervisor
cd Docker/supervisor
touch supervisord.conf
```
```
[supervisord]
nodaemon=true

[supervisorctl]

[inet_http_server]
port = 127.0.0.1:9001

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[include]
files = supervisord.d/*.conf
```
lanjut buat directory untuk masing2 service didalam image ini
```
mkdir Docker/supervisor/supervisord.d/
cd Docker/supervisor/supervisord.d/
touch php-fpm.conf laravel-scheduler.conf laravel-worker.conf
```
```
[program:php-fpm]
process_name=%(program_name)s_%(process_num)02d
command=php-fpm
autostart=true
autorestart=true
numprocs=8
user=www-data
stdout_logfile=/var/log/supervisor/php-fpm-stdout.log
stderr_logfile=/var/log/supervisor/php-fpm-stderr.log
redirect_stderr=true
```
```
[program:laravel-scheduler]
process_name=%(program_name)s_%(process_num)02d
command=/bin/sh -c "while [ true ]; do (php /var/www/artisan schedule:run --verbose --no-interaction &); sleep 60; done"
autostart=true
autorestart=true
numprocs=1
user=www-data
redirect_stderr=true
stdout_logfile=/var/log/supervisor/laravel-scheduler-stdout.log
stderr_logfile=/var/log/supervisor/laravel-scheduler-stderr.log
```
```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/artisan queue:work --sleep=3 --tries=3
autostart=true
startsecs=0
autorestart=true
numprocs=8
user=www-data
stdout_logfile=/var/log/supervisor/laravel-worker-stdout.log
stderr_logfile=/var/log/supervisor/laravel-worker-stderr.log
redirect_stderr=true
```
kita build image dg perintah
```
docker build -f Docker/dockerfile/app.Dockerfile -t app .
```
- `-f` dockerfile utk di build
- `-t` tag pada image & name. 
- `.` build path, dimana kita eksekusi docker build
![image](https://user-images.githubusercontent.com/89820226/210720859-9e897431-7794-4464-bce7-86b787d7fe5d.png)

```
touch Docker/dockerfile/web_server.Dockerfile
```
```
FROM nginx:alpine

# copy folder public dari source code laravel
COPY public /var/www/public
ADD Docker/nginx/default.conf /etc/nginx/conf.d/default.conf
WORKDIR /var/www/
```
buat konfig nginx
```
mkdir Docker/nginx
touch Docker/nginx/default.conf
```
```
server {
    listen 80;
    index index.php index.html;
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```
build image web_server dg command berikut:
```
docker build -f Docker/dockerfile/web_server.Dockerfile -t web_server .
```
cek images yg sudah berhasil kita build
```
docker images
# atau
docker image ls
```
![image](https://user-images.githubusercontent.com/89820226/210722972-7b488fe2-bbde-4684-a86c-52d8bfa6dd2f.png)

#### 4. Push Images ke Repository

langkah push image ke docker hub:
1. login dg akun docker hub
2. tag image dg format `username/repository-name:tag-name-id`, misal `whypeout/laravel-app:20230105` dan `whypeout/laravel-web_server:20230105`
3. push: upload image ke docker hub

```
docker login

docker tag app budasuyasa/laravel-prod-demo-app:prod
docker push budasuyasa/laravel-prod-demo-app:prod

docker tag web_server budasuyasa/laravel-prod-demo-web_server:prod
docker push budasuyasa/laravel-prod-demo-web_server:prod
```

#### 5. Pull docker image

berpindah ke production server via ssh, pastikan server sudah terinstall docker engine. jika repository bersifat privat, kita perlu login dg `docker login`. kita pull image:
```
docker pull budasuyasa/laravel-prod-demo-app:prod
docker pull budasuyasa/laravel-prod-demo-web_server:prod
```

#### 6. Menyiapkan docker-compose.yml

pada production server, kita buat direktori kerja baru. misal pada $HOME directory.
```
mkdir ~/laravel-prod && cd ~/laravel-prod
touch docker-compose.yml
```
```yaml
version: '3'
services:
  app:
    image: budasuyasa/laravel-prod-demo-app:prod
    container_name: app
    restart: unless-stopped
    tty: true
    env_file: .env
    depends_on:
      - cache
    networks:
      - laravel-prod
    
  web_server:
    image: budasuyasa/laravel-prod-demo-web_server:prod
    container_name: web_server
    restart: unless-stopped
    tty: true
    ports:
      - "8002:80"
    depends_on:
      - app
    networks:
      - laravel-prod
  
  cache:
    image: redis:alpine
    container_name: cache
    restart: unless-stopped
    networks:
      - laravel-prod
      
networks:
  laravel-prod:
    driver: bridge
```





















