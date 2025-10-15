# Docker Guide - Network

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 9/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/userguide/networking/


## List current network
docker network ls

ip addr show

###### The default bridge network
#### Examine the bridge network
docker network inspect bridge

#### Start 2 busybox container connected to the default bridge network
docker run -itd --name=container1 busybox

docker run -itd --name=container2 busybox

## Examine how network looks from inside the container
docker attach container1
ip -4 addr


###### User-defined networks
#### Bridge Network
## Create your own bridge network
docker network ls
docker network create --driver bridge my_bridge_net

docker network ls

docker run --network=my_bridge_net -itd --name=container3 busybox

docker network inspect my_bridge_net

###### Exposing and publishing ports
## Publish port 80 on nginx container to a random high port (higher than 30000)
docker run -it -d -p 80 nginx
docker ps

## Publish port 80 on nginx container to port 8080 on host machine
docker run -it -d -p 8080:80 nginx
docker ps

###### Set static ip for container
#### Option 1: From Docker CLI
docker network create --subnet=172.20.0.0/16 mynet123
docker run --net mynet123 --ip 172.20.0.99 -p 8080:80 --hostname <container_hostname> -it -d nginx

curl 172.20.0.99:80
or
curl localhost:8080

#### Option 2: In docker-compose.yml
vi docker-compose.yml
**Content**
version: '3'
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



docker-compose up -d
docker ps
docker network ls
curl 192.168.0.99
