# Containerize CodeIgniter (Apache MySQL PHP Xdebug)

sources:
- docker on macos [https://torbjornzetterlund.com/apache-mysql-php-xdebug-and-codeigniter-on-macos-using-docker/](https://torbjornzetterlund.com/apache-mysql-php-xdebug-and-codeigniter-on-macos-using-docker/)

```
#Dockerfile
FROM php:8.0-apache

RUN apt update
RUN apt install -y wget vim git zip unzip zlib1g-dev libzip-dev libpng-dev

RUN docker-php-ext-install mysqli pdo_mysql gd zip pcntl exif
RUN docker-php-ext-enable mysqli

RUN a2enmod headers expires rewrite

RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
# RUN pecl install xdebug-3.1.6 for php7.4
ENV PHP_IDE_CONFIG 'serverName=www.al.pt'

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

WORKDIR /var/www/html
```

```
# build image
docker image build --no-cache -f docker/Dockerfile -t dlamp .

# run image as container
docker run -d -p 80:80 dlamp

# check running container
docker container ps
```

```
# full stack docker-compose.yml
version: 3.7

x-defaults:
  network: &network
    networks:
      - net
      
services:
  php:
    image: dlamp
    depends_on:
      - db
    ports:
      - 8282:80
    volumes:
      - .:/var/www/html
    configs:
      - source: apache-vhosts
        target: /etc/apache2/sites-available/000-default.conf
      - source: php-ini
        target: /usr/local/etc/php/conf.d/local.ini
    <<: *network
    
  db:
    platform: linux/amd64
    image: mysql/mysql-server:8.0.23
    container_name: mysql-container
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=dlamp
      - MYSQL_USER=toor
      - MYSQL_PASSWORD=toor
    volumes:
      - ./localhost.sql:/docker-entrypoint-initdb.d/localhost.sql
      - ./privileges.sql:/docker-entrypoint-initdb.d/privileges.sql
    <<: *network
    
  phpmyadmin:
    image: phpmyadmin:latest
    depends_on:
      - db
    restart: always
    environment:
      - PMA_ARBITRARY=1
    ports:
      - 8888:80
    <<: *network
    
networks:
  net:
  
volumes:
  data:
    external: true
```

```
docker stack deploy -c docker-compose.yml dev
```



















































