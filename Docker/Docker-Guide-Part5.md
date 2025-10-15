# Docker Guide Part 5: Stacks

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-04
- **Tested on:** CentOS 7

Extend the Swarm deployment by adding a visualizer service and persistent Redis
backend to the stack.

## Start the Swarm

```bash
docker-machine ls
docker-machine start myvm1
docker-machine start myvm2
docker-machine ssh myvm1 "docker node ls"
```

## Compose file with visualizer

```yaml
version: "3"
services:
  web:
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
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

## Deploy the stack

```bash
docker-machine env myvm1
eval "$(docker-machine env myvm1)"
docker stack deploy -c docker-compose.yml getstartedlab
docker stack ps getstartedlab
```

Access the visualizer at `http://<manager-ip>:8080`.

## Add Redis for persistent state

Update `docker-compose.yml` to include Redis:

```yaml
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
```

Create the data directory on the manager and redeploy.

```bash
docker-machine ssh myvm1 "mkdir -p /home/docker/data"
docker stack deploy -c docker-compose.yml getstartedlab
docker service ls
```

Visit `http://<manager-ip>` to validate the front-end and confirm Redis persists
state between restarts.
