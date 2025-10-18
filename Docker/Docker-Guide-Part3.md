# Docker Guide Part 3: Services

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-03
- **Tested on:** CentOS 7

Deploy a replicated service using Docker Swarm and Docker Compose.

## Prerequisites

Install Docker Compose 1.18.0 (or newer):

```bash
sudo curl -L \
  https://github.com/docker/compose/releases/download/1.18.0/docker-compose-"$(uname -s)"-"$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Ensure Docker Engine is installed and Swarm mode is available.

## Compose file

```yaml
version: "3"
services:
  web:
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
```

Replace the image reference with your Docker Hub repository.

## Deploy the stack

```bash
docker swarm init
docker stack deploy -c docker-compose.yml getstartedlab
docker service ls
docker service ps getstartedlab_web
for i in {1..5}; do curl -4 http://localhost; done
```

## Tear everything down

```bash
docker stack rm getstartedlab
docker swarm leave --force
```
