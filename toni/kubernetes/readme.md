# Kumpulan Docker Image & Dokumentasi nya

## BITNAMI

- github bitnami containers [https://github.com/bitnami/containers](https://github.com/bitnami/containers)
- wordpress nginx [https://hub.docker.com/r/bitnami/wordpress-nginx/](https://hub.docker.com/r/bitnami/wordpress-nginx/)

```
curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/wordpress/docker-compose.yml > docker-compose.yml
docker-compose up
```


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
