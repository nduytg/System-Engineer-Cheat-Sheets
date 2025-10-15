# Docker Guide - Volume

- **Author:** nduytg
- **Version:** 0.9
- **Date:** 5/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/admin/volumes/volumes/#choose-the--v-or-mount-flag
http://training.play-with-docker.com/docker-volumes/

###### Create and manage volumes
docker volume create my-vol

## List volumes
docker volume ls

## Insprect a volume
docker volume inspect my-vol

## Remove a volume
docker volume rm my-vol

###### Start a container with a volume
#### Option 1: --mount
docker run -d \
	-it \
	--name devtest \
	--mount source=myvol2,target=/app \
	nginx:latest

#### Option 2: -v
docker run -d \
	-it \
	--name devtest \
	-v myvol2:/app \
	nginx:latest

docker inspect devtest

**Result**
"Mounts": [
            {
                "Type": "volume",
                "Name": "myvol2",
                "Source": "/var/lib/docker/volumes/myvol2/_data",
                "Destination": "/app",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],


## Stop the container
docker container stop devtest
docker container rm devtest

## Remove the volume
docker volume rm myvol2

###### Start a service with volumes
## Start a swarm
docker swarm init

## Start the service
docker service create -d \
	--replicas=4 \
	--name devtest-service \
	--mount source=myvol2,target=/app \
	nginx:latest

## Verify that the service is running
docker service ps devtest-service

## Verify if the volume exists
docker volume ls

## Remove the serivce
docker service rm devtest-service

## Verify if the volume still exists (it do!!)
docker volume ls

###### Populate a volume using a container
## Option 1: --mount
docker run -d \
  -it \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest

## Option 2: -v
docker run -d \
	-it \
	--name=nginxtest \
	-v nginx-vol:/usr/share/nginx/html \
	nginx:latest

## Check the contents of volume
ls /var/lib/docker/volumes/nginx-vol/_data

## Clean up the container and volume
docker container stop nginxtest

docker container rm nginxtest

docker volume rm nginx-vol

###### Read-only volume
## Option 1: --mount
docker run -d \
	-it \
	--name=nginxtest \
	--mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
	nginx:latest

## Option 2: -v
docker run -d \
	-it \
	--name=nginxtest \
	-v nginx-vol:/usr/share/nginx/html:ro \
	nginx:latest

**Result**
"Mounts": [
            {
                "Type": "volume",
                "Name": "nginx-vol",
                "Source": "/var/lib/docker/volumes/nginx-vol/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": false,
                "Propagation": ""
            }
        ],


## Clean up the container and volume
docker container stop nginxtest

docker container rm nginxtest

docker volume rm nginx-vol

###### Use a volume driver
...(not yet)...
