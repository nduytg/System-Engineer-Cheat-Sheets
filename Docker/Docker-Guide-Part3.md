# Docker Guide - Part 3: Services

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 3/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/installation/linux/docker-ce/centos/
https://docs.docker.com/get-started/
https://docs.docker.com/compose/install/

#### Prerequisites
## Install Docker Compose 1.18.0
curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version

#### docker-compose.yml file
vi docker-compose.yml
**Content**
version: "3"
services:
  web:
    replace username/repo:tag with your name and image details
    image: nduytg/get-started:part2
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:



#### Run new load-balanced app
docker swarm init
docker stack deploy -c docker-compose.yml getstartedlab
docker service ls
docker service ps getstartedlab_web

for i in {1..5} ; do curl -4 http://localhost ; done

#### Take down the app and the swarm
docker stack rm getstartedlab

docker swarm leave --force
