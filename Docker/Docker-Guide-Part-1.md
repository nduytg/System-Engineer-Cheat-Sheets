# Docker Guide - Part 1: Install Docker

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/installation/linux/docker-ce/centos/
https://docs.docker.com/get-started/

#### Docker requirements
## Kernel version 3.10 or higher (CentOS7, Ubuntu 16,...)
## 64-bit OS

###### Remove old version of docker
yum remove docker docker-common docker-selinux docker-engine

## Remove all old images, containers and volumes
rm -rf /var/lib/docker

###### Set up Docker Repository
## Install prerequisite packages
yum install -y yum-utils device-mapper-persistent-data lvm2

## Enable "stable" docker
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

## Enable the "edge" and "test" repo for experiment
yum-config-manager --enable docker-ce-edge
yum-config-manager --enable docker-ce-test

###### Install Docker
yum install docker-ce

## Start Docker
systemctl start docker
systemctl enable docker

## Check Docker version
[root@centos-docker ~]# docker --version
Docker version 17.12.0-ce, build c97c6d6

## Run hello-world image on Docker
docker run hello-world

## List local images
docker images

## Searching images (on default registry)
docker search centos
## Pull specific images
docker pull centos:centos6
docker pull centos
docker images

docker run [name|image-id]

## Inspect local images
docker inspect centos
docker inspect hello-world

## List running containers
docker ps

## List all containers
docker ps -a

run docker interactive, terminal
docker run -it centos:latest

run docker as deamon
docker run -d centos:lastest
