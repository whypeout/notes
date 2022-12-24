# Docker in Docker

sources:
- [https://shisho.dev/blog/posts/docker-in-docker/](https://shisho.dev/blog/posts/docker-in-docker/)

```
docker run --privileged --name dind -d docker:stable
docker exec -it dind /bin/ash

#or
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock:ro docker /bin/ash

#docker in jenkins
volumes:
- ${HOST_DOCKER}:/var/run/docker.sock
- {$HOST_JENKINS_DATA}:/var/jenkins_home

# docker in gitlab
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

```
