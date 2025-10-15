# Docker Guide - tmpfs Mount

- **Author:** nduytg
- **Version:** 0.1
- **Date:** 5/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/admin/volumes/bind-mounts/
http://training.play-with-docker.com/docker-volumes/


###### Start a container with a bind mount
## Option 1: --mount
docker run -d \
	-it \
	--name devtest \
	--mount type=bind

## Option 2: -v
