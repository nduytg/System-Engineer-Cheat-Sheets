# Docker Guide - Part 4: Stacks

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 4/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/installation/linux/docker-ce/centos/
https://docs.docker.com/get-started/
https://docs.docker.com/machine/install-machine/


###### Boot up your swarm
docker-machine ls
docker-machine start myvm1
docker-machine start myvm2

docker-machine ssh myvm1 "docker node ls"

###### Edit docker-compose.yml
vi docker-compose.yml
**Content**
version: "3"
services:
  web:
    replace username/repo:tag with your name and image details
    image: nduytg/get-started:part2
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:


## Configure a docker-machine shell to the swarm manager
docker-machine env myvm1
eval $(docker-machine env myvm1)

## Deploy your stack
docker stack deploy -c docker-compose.yml getstartedlab

## Go to http://<your_ip>:8080
## Or
docker stack ps getstartedlab

###### Update docker-compose.yml with Redis (storing app data)
vi docker-compose.yml
**Content**
version: "3"
services:
  web:
    replace username/repo:tag with your name and image details
    image: nduytg/get-started:part2
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - /home/docker/data:/data
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:


Create a ./data directory on the manager:
docker-machine ssh myvm1 "mkdir ./data"

docker stack deploy -c docker-compose.yml getstartedlab

docker service ls

## Goto http://<your_ip>
