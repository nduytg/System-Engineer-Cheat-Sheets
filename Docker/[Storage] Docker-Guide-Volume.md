# Docker Guide: Volumes

- **Author:** nduytg
- **Version:** 0.9
- **Date:** 2018-01-05
- **Tested on:** CentOS 7

Docker volumes provide persistent storage managed by the Docker engine. This
reference covers creation, usage with containers and services, and read-only
mounts.

## Create and manage volumes

```bash
docker volume create my-vol
docker volume ls
docker volume inspect my-vol
docker volume rm my-vol
```

## Attach a volume to a container

### Using `--mount`

```bash
docker volume create myvol2
docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

### Using `-v`

```bash
docker run -d \
  --name devtest-v \
  -v myvol2:/app \
  nginx:latest
```

Inspect the container to confirm the mount point:

```bash
docker inspect devtest --format '{{ json .Mounts }}' | jq
```

Stop and remove the container when finished:

```bash
docker container stop devtest devtest-v
docker container rm devtest devtest-v
docker volume rm myvol2
```

## Use a volume with a Swarm service

```bash
docker volume create myvol2
docker swarm init
docker service create -d \
  --replicas 4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
docker service ps devtest-service
docker volume ls
docker service rm devtest-service
docker volume rm myvol2
```

## Populate a volume

```bash
docker run -d \
  --name nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
ls /var/lib/docker/volumes/nginx-vol/_data
docker container stop nginxtest
docker container rm nginxtest
docker volume rm nginx-vol
```

Repeat the same steps with the `-v` flag if you prefer the short syntax.

## Read-only volumes

```bash
docker volume create nginx-vol
docker run -d \
  --name nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
docker inspect nginxtest --format '{{ json .Mounts }}' | jq
docker container stop nginxtest
docker container rm nginxtest
docker volume rm nginx-vol
```

To use the short syntax, replace the `--mount` flag with
`-v nginx-vol:/usr/share/nginx/html:ro`.
