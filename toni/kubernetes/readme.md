# Kumpulan Docker Image & Dokumentasi nya

## BITNAMI

- github bitnami containers [https://github.com/bitnami/containers](https://github.com/bitnami/containers)
- wordpress nginx [https://hub.docker.com/r/bitnami/wordpress-nginx/](https://hub.docker.com/r/bitnami/wordpress-nginx/)

```
curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/wordpress/docker-compose.yml > docker-compose.yml
docker-compose up

#atau
docker pull bitnami/wordpress-nginx:latest
docker pull bitnami/wordpress-nginx:TAG

#via git
git clone https://github.com/bitnami/containers.git
cd bitnami/APP/VERSION/OPERATING-SYSTEM
docker build -t bitnami/APP:latest .

# menjalankan image via cmd line
docker network create wordpress-network
docker volume create --name mariadb_data
docker run -d --name=mariadb \
  -e ALLOW_EMPTY_PASSWORD=yes \
  -e MARIADB_USER=bn_wordpress \
  -e MARIADB_PASSWORD=bitnami \
  -e MARIADB_DATABASE=bitnami_wordpress \
  --network wordpress-network \
  --volume mariadb_data:/bitnami/mariadb \
  bitnami/mariadb:latest
docker volume create --name wordpress_data
docker run -d --name wordpress \
  -p 8080:8080 -p 8443:8443 \
  --env ALLOW_EMPTY_PASSWORD=yes \
  -e WORDPRESS_DATABASE_USER=bn_wordpress \
  -e WORDPRESS_DATABASE_PASSWORD=bitnami \
  -e WORDPRESS_DATABASE_NAME=bitnami_wordpress \
  --net wordpress-network \
  -v wordpress_data:/bitnami/wordpress \
  bitnami/wordpress-nginx:latest
  
# modify docker-compose.yml to use host directory to mount inside container
-   - mariadb_data:/bitnami/mariadb
+   - /path/to/mariadb-persistence:/bitnami/mariadb
-   - wordpress_data:/bitnami/wordpress
+   - /path/to/wordpress-persistence:/bitnami/wordpress

# note: image bitnami berjalan sbg non-root container, pastikan permission utk UID `1001`
```

KEY | Details
----|------------
NGINX_HTTP_PORT_NUMBER  | 8080; port http nginx
NGINX_HTTPS_PORT_NUMBER | 8443; port https nginx
NGINX_ENABLE_ABSOLUTE_REDIRECT | no; absolute URL di location header pada redirection
NGINX_ENABLE_PORT_IN_REDIRECT | no; listen port for redirection issued by NGINX
WORDPRESS_USERNAME | user; login username wordpress
WORDPRESS_PASSWORD | bitnami; wordpress login password
WORDPRESS_EMAIL | user@example.com; wp app email
WORDPRESS_FIRST_NAME | FirstName; user first name
WORDPRESS_LAST_NAME | LastName; user last name
WORDPRESS_BLOG_NAME | user's blog; wp blog name
WORDPRESS_DATA_TO_PERSIST | " ", "wp-config.php wp-content";space separated list of file & dir to persist
WORDPRESS_RESET_DATA_PERMISSIONS | no; force resetting ownership/permission on persited data ketika wp restart
WORDPRESS_TABLE_PREFIX | none; all; wp plugins to install & activate, separated commas
WORDPRESS_EXTRA_INSTALL_ARGS | extra flags to append to wp 'wp core install'
WORDPRESS_ENABLE_HTTPS | no; use https by default
WORDPRESS_SKIP_BOOTSTRAP | no; skip wp install wizzard. perlu utk databse dg existing wp data
WORDPRESS_ENABLE_REVERSE_PROXY | no; enable wp support reverse proxy headers

MULTISITE | Details
----------|------------
WORDPRESS_ENABLE_MULTISITE | no; enable wp multisite
WORDPRESS_MULTISITE_HOST | wp hostname/address
WORDPRESS_MULTISITE_EXTERNAL_HTTP_PORT_NUMBER | 80;
WORDPRESS_MULTISITE_EXTERNAL_HTTPS_PORT_NUMBER | 443;
WORDPRESS_MULTISITE_EXTERNAL_NETWORK_TYPE | subdomain; subdirectory, subfolder
WORDPRESS_MULTISITE_ENABLE_NIP_IO_REDIRECTION | no;
WORDPRESS_MULTISITE_FILEUPLOAD_MAXK | 81920 kilobytes max uploads

database | Details
----------|------------
WORDPRESS_DATABASE_HOST | mariadb; mysql
WORDPRESS_DATABASE_PORT_NUMBER | 3306
WORDPRESS_DATABASE_NAME | bitnami_wordpress
WORDPRESS_DATABASE_USER | bn_wordpress
WORDPRESS_DATABASE_PASSWORD | empty
WORDPRESS_DATABASE_DATABASE_SSL | no; ssl database connection
WORDPRESS_DATABASE_DATABASE_SSL_CERT/KEY/CA_FILE | path to file
ALLOW_EMPTY_PASSWORD | no; blank password

SMTP | Details
----------|------------
WORDPRESS_SMTP_HOST | smtp host; smtp.gmail.com
WORDPRESS_SMTP_PORT | smtp port; 587
WORDPRESS_SMTP_USER | account user; user@gmail.com
WORDPRESS_SMTP_PASSWORD | account password; my_password

PHP | Details
----------|------------
PHP_ENABLE_OPCACHE | yes; opcache
PHP_EXPOSE_PHP | http header w php version
PHP_MAX_EXECUTION_TIME | php max exec time 
PHP_MAX_INPUT_TIME | max input time 
PHP_MAX_INPUT_VARS | max amount input var
PHP_MEMORY_LIMIT | 256M memory limit
PHP_POST_MAX_SIZE | max POST PHP size
PHP_UPLOAD_MAX_FILESIZE | php uploads



---

## Docker Containerize NodeJS Application

[https://docs.docker.com/get-started/02_our_app/](https://docs.docker.com/get-started/02_our_app/)

```
# clone repo :: getting-started/app/package.json
git clone https://github.com/docker/getting-started.git

# build container app image;; nano getting-started/app/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000

# build
cd getting-started/app
docker build -t getting-started .

# start app container
docker run -d -p 3000:3000 getting-started

# open with browser http://localhost:3000


```
