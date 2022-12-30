# Dockerize & Deploy Laravel App ke Production

sources:
- [https://www.koyeb.com/tutorials/dockerize-and-deploy-a-laravel-application-to-production](https://www.koyeb.com/tutorials/dockerize-and-deploy-a-laravel-application-to-production)
- [https://help.clouding.io/hc/en-us/articles/360010679999-How-to-Deploy-Laravel-with-Docker-on-Ubuntu-18-04](https://help.clouding.io/hc/en-us/articles/360010679999-How-to-Deploy-Laravel-with-Docker-on-Ubuntu-18-04)

# Prereq
```
apt update -y
apt upgrade -y
apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_relase -css) stable"
apt update -y
apt install docker-ce -y
apt install docker-compose -y
systemctl status docker
```
# Buat Laravel App Baru
```
composer create-project --prefer-dist laravel/laravel laravel-demo
php artisan serve
# buka browser ke http://localhost:8000

git clone https://github.com/laravel/laravel.git laravel
cd laravel
docker run --rm -v $PWD:/app composer install
```

# Dockerize Laravel App
kita pakai image `webdevops/php-nginx:7.4-alpine` sbg base image, sudah termasuk Nginx dg PHP-FPM.
```
# nano Dockerfile
FROM webdevops/php-nginx:7.4-alpine
RUN apk add oniguruma-dev postgresql-dev libxml2-dev
RUN docker-php-ext-install bcmath ctype fileinfo json mbstring pdo_mysql pdo_pgsql tokenizer xml
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV WEB_DOCUMENT_ROOT /app/public
ENV APP_ENV production
WORKDIR /app
COPY . .
RUN composer install --no-interaction --optimize-autoloader --no-dev
RUN php artisan config:cache
RUN php artisan route:cache
RUN php artisan view:cache
RUN chown -R application:application .
```
```
docker build -t username/laravel-demo .
docker run --rm -it -p 9000:80 username/laravel-demo
docker push username/laravel-demo
```

---

# Docker compose style

```yaml
# nano laravel/docker-compose.yml
version: '3'
services:
  app: 
    build:
      context: .
      dockerfile: Dockerfile
    image: digitalocean.com/php
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    volumes:
      - ./:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - app-network
      
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - app-network
      
  db:
    image: mysql:5.7.22
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - 3306:3306
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: password
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - dbdata:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - app-network
      
networks:
  app-network:
    driver: bridge
    
volumes:
  dbdata:
    driver: local
```

```dockerfile
# nano laravel/Dockerfile
FROM php:7.2-fpm
COPY composer.lock composer.json /var/www/
WORKDIR /var/www
RUN apt update && apt install -y build-essential mysql-client libpng-dev libjpeg62-turbo-dev libfreetype6-dev locales zip jpegoptim optipng pngquant gifsicle vim unzip git nano curl
RUN apt clean && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-install gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www
COPY . /var/www
COPY --chown=www:www . /var/www
USER www
EXPOSE 9000
CMD ["php-fpm"]
```
## Konfigurasi PHP, MySQL dan NGINX utk Laravel
```
mkdir laravel/php
nano laravel/php/local.ini

#tambahkan baris ini
# upload_max_filesize=40M
# post_max_size=40M

mkdir -p laravel/nginx/conf.d
nano laravel/nginx/conf.d/app.conf
#tambahkan baris ini
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

mkdir laravel/mysql
nano laravel/mysql/my.cnf
#tambahkan berikut
[mysqld]
general_log = 1
general_log_file = /var/lib/mysql/general.log
```
## Deploy Laravel dg NGINX & MySQL Services

```
cp .env.example .env
docker-compose up -d
docker ps
```

## Konfigurasi laravel container

```
docker-compose exec app nano .env
#ubah bagian berikut
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=password

# generate laravel application key & clear cache
docker-compose exec app php artisan key:generate
docker-compose exec app php artisan config:cache

# migrate database
docker-compose exec app php artisan migrate
```
## Akses laravel web interface
buka browser ke `http://alamat-ip-server`























