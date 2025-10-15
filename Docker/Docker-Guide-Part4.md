# Docker Guide - Part 4: Swarm

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 3/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/installation/linux/docker-ce/centos/
https://docs.docker.com/get-started/
https://docs.docker.com/machine/install-machine/
https://wiki.centos.org/HowTos/Virtualization/VirtualBox

###### Prerequisites
#### Install Docker Machine
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && \
chmod +x /tmp/docker-machine && cp /tmp/docker-machine /usr/local/bin/docker-machine

docker-machine version

#### Install VirtualBox
cd /etc/yum.repos.d
wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

## Install DKMS (Dynamic Kernel Module) for VirtualBox
yum --enablerepo=epel install dkms
yum groupinstall "Development Tools"
yum install kernel-devel

## Install VirtualBox 5.2
yum install VirtualBox-5.2.x86_64

## Recompile kernel module
sudo /sbin/vboxconfig

#### Set up your Swarm
## Create a cluster
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

## Initialize the swarm and add nodes
docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"

Swarm initialized: current node (fon41sug27y9mwn3asfqo3ork) is now a manager.
To add a worker to this swarm, run the following command:

docker swarm join --token SWMTKN-1-5c0ya8i59hs18zvc30z7ssnbsr3vq709edg7loc2rydia3oksf-8wu15o42wti2k1pcafk86vdvm 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

## Join the swarm
docker-machine ssh myvm2 "docker swarm join --token SWMTKN-1-5c0ya8i59hs18zvc30z7ssnbsr3vq709edg7loc2rydia3oksf-8wu15o42wti2k1pcafk86vdvm 192.168.99.100:2377"

## List the nodes in the swarm
docker-machine ssh myvm1 "docker node ls"

## Deploy app on the swarm cluster
docker-machine ssh myvm1 "docker node ls"

## Configure a docker-machine shell to the swarm manager
docker-machine env myvm1
eval $(docker-machine env myvm1)

docker stack deploy -c docker-compose.yml getstartedlab

## Test app
for i in {1..5} ; do curl -4 http://<myvm_ip> ; done

## Remove the stack
docker stack rm getstartedlab

## Unsetting docker-machine shell
eval $(docker-machine env -u)

## Stop/Start docker machines
docker-machine ls

docker-machine start/stop <machine-name>
