# Docker Guide Part 4: Swarm

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-03
- **Tested on:** CentOS 7

Use Docker Machine and VirtualBox to provision a two-node Swarm cluster, deploy a
stack, and tear it down.

## Prerequisites

Install Docker Machine:

```bash
sudo curl -L \
  https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-"$(uname -s)"-"$(uname -m)" \
  -o /usr/local/bin/docker-machine
sudo chmod +x /usr/local/bin/docker-machine
docker-machine version
```

Install VirtualBox and its dependencies:

```bash
cd /etc/yum.repos.d
sudo wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
sudo yum --enablerepo=epel install -y dkms
sudo yum groupinstall -y "Development Tools"
sudo yum install -y kernel-devel
sudo yum install -y VirtualBox-5.2
sudo /sbin/vboxconfig
```

## Provision the Swarm

```bash
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

Initialize the cluster from the first node:

```bash
docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
```

Join the worker:

```bash
docker-machine ssh myvm2 "docker swarm join --token <worker-token> <manager-ip>:2377"
```

List cluster members:

```bash
docker-machine ssh myvm1 "docker node ls"
```

## Deploy a stack

Configure your shell to talk to the manager and launch the stack described in
`docker-compose.yml` from Part 3.

```bash
docker-machine env myvm1
eval "$(docker-machine env myvm1)"
docker stack deploy -c docker-compose.yml getstartedlab
for i in {1..5}; do curl -4 http://<myvm_ip>; done
docker stack rm getstartedlab
eval "$(docker-machine env -u)"
```

## Manage machines

```bash
docker-machine ls
docker-machine stop <machine-name>
docker-machine start <machine-name>
```
