# PHP 7.4 apache xdebug pgsql

```dockerfile
FROM php:7.4-apache

RUN apt update
RUN apt install -y wget vim git zip unzip zlib1g-dev libzip-dev libpng-dev libpq-dev

RUN docker-php-ext-install mysqli pdo_mysql gd zip pcntl exif pgsql pdo_pgsql
RUN docker-php-ext-enable mysqli pgsql

RUN a2enmod headers expires rewrite

RUN pecl install xdebug-3.1.6
RUN docker-php-ext-enable xdebug
ENV PHP_IDE_CONFIG 'serverName=www.al.pt'

COPY --from=composer:2 /usr/bin/composer /usr/lib/bin/composer

WORKDIR /var/www/html
```

# PHP 8 apache xdebug pgsql

```dockerfile
FROM php:8.0-apache

RUN apt update
RUN apt install -y wget vim git zip unzip zlib1g-dev libzip-dev libpng-dev libpq-dev

RUN docker-php-ext-install mysqli pdo_mysql gd zip pcntl exif pgsql pdo_pgsql
RUN docker-php-ext-enable mysqli pgsql

RUN a2enmod headers expires rewrite

RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
ENV PHP_IDE_CONFIG 'serverName=www.al.pt'

COPY --from=composer:2 /usr/bin/composer /usr/lib/bin/composer

WORKDIR /var/www/html
```
