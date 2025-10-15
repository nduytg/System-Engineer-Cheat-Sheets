# Docker Guide Part 1: Install Docker Engine

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-02
- **Tested on:** CentOS 7

This section follows the official Docker Engine installation steps for CentOS 7.

## Requirements

- 64-bit operating system
- Linux kernel 3.10 or later

Reference documentation:

- <https://docs.docker.com/engine/install/centos/>
- <https://docs.docker.com/get-started/>

## Remove legacy versions

```bash
sudo yum remove docker docker-common docker-selinux docker-engine
sudo rm -rf /var/lib/docker
```

## Configure the Docker repository

Install prerequisites and enable the stable repository. Uncomment the final two
commands if you wish to opt in to the edge or test channels.

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
# sudo yum-config-manager --enable docker-ce-edge
# sudo yum-config-manager --enable docker-ce-test
```

## Install and start Docker

```bash
sudo yum install -y docker-ce
sudo systemctl enable --now docker
```

## Verify the installation

```bash
docker --version
docker run hello-world
docker images
docker search centos
docker pull centos:centos6
docker pull centos
docker inspect centos
docker inspect hello-world
docker ps
docker ps -a
docker run -it centos:latest
docker run -d centos:latest
```

Use `docker run IMAGE` (with either the image name or ID) to start additional
containers as required.
