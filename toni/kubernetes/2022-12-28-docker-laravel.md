## Laravel Docker
sources: [https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-set-up-laravel-with-docker-compose-on-ubuntu-20-04)

## 1. download demo app
demo app: 
```sh
cd ~ && curl -L https://github.com/do-community/travellist-laravel-demo/archive/tutorial-1.0.1.zip -o travellist.zip
sudo apt update && sudo apt install unzip && unzip travellist.zip && move travellist-laravel-demo* travellist-demo && cd travellist-demo && cp .env.example .env && nano .env
```
## 2. env
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
## 3.dockerfile
```dockerfile
FROM php:7.4-fpm
ARG user
ARG uid
RUN apt update && apt install -y git curl \
    libpng-dev libonig-dev libxml2-dev zip unzip
RUN apt clean && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && chown -R $user:$user /home/$user 
WORKDIR /var/www
USER $user 
```
## setup nginx config & database dump files
```bash
mkdir -p docker-compose/nginx
nano docker-compose/nginx/travellist.conf
```
```conf
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
```
mkdir docker-compose/mysql
nano docker-compose/mysql/init_db.sql
```
```sql
DROP TABLE IF EXISTS `places`;
CREATE TABLE places (id bigint(20) unsigned NOT NULL AUTO_INCREMENT, name varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL, visited tinyint(1) NOT NULL DEFAULT '0', PRIMARY KEY (id)) ENGINE=innoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
INSERT INTO places (name,visited) VALUES('Berlin',0),('Budapest',0),('Cincinnati',1),('Denver',0),('Helsinki',0),('Lisbon',0),('Moscow',1),('Nairobi',0),('Oslo',1),('Rio',0),('Tokyo',0);
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
```
docker-compose build app
docker-compose up -d
docker-compose ps
docker-compose exec app ls -l
docker-compose exec app composer install
docker-compose exec app php artisan key:generate
# curl http://domain-ip:8000
docker-compose logs nginx
docker-compose pause
docker-compose unpause
docker-compose down
```
