# Docker Guide: Networking

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-09
- **Tested on:** CentOS 7

Explore Docker networking fundamentals including default bridges, user-defined
networks, port publishing, and static IP assignments.

## Inspect existing networks

```bash
docker network ls
ip addr show
```

## Default bridge network

Inspect the default bridge and run two containers that share it.

```bash
docker network inspect bridge
docker run -itd --name container1 busybox
docker run -itd --name container2 busybox
docker attach container1
ip -4 addr
exit
```

## User-defined bridge network

```bash
docker network create --driver bridge my_bridge_net
docker run --network my_bridge_net -itd --name container3 busybox
docker network inspect my_bridge_net
```

## Publish container ports

```bash
docker run -d -p 80 nginx        # random host port
docker ps
docker run -d -p 8080:80 nginx   # explicit host port
docker ps
```

## Assign static IP addresses

### Docker CLI

```bash
docker network create --subnet 172.20.0.0/16 mynet123
docker run --net mynet123 --ip 172.20.0.99 -p 8080:80 \
  --hostname <container_hostname> -d nginx
curl 172.20.0.99:80
curl localhost:8080
```

### Docker Compose

```yaml
version: "3"
services:
  nginx:
    image: nginx
    container_name: my-nginx
    networks:
      mynet123:
        ipv4_address: 192.168.0.99
networks:
  mynet123:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.0.0/24
```

```bash
docker-compose up -d
docker ps
docker network ls
curl 192.168.0.99
```
