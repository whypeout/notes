## Laravel Docker
sources: [https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04)

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
