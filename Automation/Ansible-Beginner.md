# Learning Ansible

- **Author:** nduytg
- **Version:** 0.3
- **Date:** 4/12/17
- **Tested on:** Ubuntu 16.04

## Ansible + LXC on Ubuntu

#### Install Ansible
### Option 1: Clone from git
apt-get install git-core

cd ~
git clone https://github.com/ansible/ansible

cd ansible
git log
git submodule update --init --recursive
apt-get install python-jinja2 python-paramiko python-yaml sshpass

which ansible

less ./hacking/env-setup
source ./hacking/env-setup

which ansible
ansible --version

### Option 2: Install form Yum
apt-get update
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible

#### Create Target Machine with LXC
## Simulate multiple server with LXC (Linux Container)
apt-get update && apt-get install lxc

lxc-ls --fancy

## 2 Web, 1 DB
lxc-create -n web1 -t ubuntu
lxc-create -n web2 -t ubuntu
lxc-create -n db1 -t ubuntu

## List all containers
lxc-ls -f

## Start these containers as deamon processes
lxc-start -n web1 -d

## Log into web1 container
lxc-attach -n web1

#### Prepare target machine (LXC Containers)
- Requirement: Python 2 and OpenSSH
lxc-attach -n web1
apt-get install python-minimal

#### Ad-hoc commands
vim inventory
[allservers]
10.0.3.113
10.0.3.247
10.0.3.95

[web]
10.0.3.247
10.0.3.95

[database]
10.0.3.113
---

## Ping all servers named in inventory list
ansible allservers -m ping -u ubuntu -i inventory

## Show available memory on all servers
ansible allservers -a "free -h" -u ubuntu -i inventory

## Check if these hosts have nginx installed on
ansible allservers -a "apt list installed | grep nginx" -u ubuntu -i inventory

## Update all hosts
ansible allservers -a "apt-get update" -u root -i inventory

## Install nginx on web1 and web2
ansible web -a "apt-get install -y nginx" -u root -i inventory

## Restart nginx on web1 and web2
ansible web -m service -a "name=nginx state=restarted" -i inventory -u root
