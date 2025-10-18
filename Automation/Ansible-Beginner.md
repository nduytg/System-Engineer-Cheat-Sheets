# Learning Ansible

- **Author:** nduytg
- **Version:** 0.3
- **Date:** 2017-04-12
- **Tested on:** Ubuntu 16.04

This quick-start shows how to install Ansible on Ubuntu and exercise it against
lightweight LXC containers that simulate a small fleet of hosts.

## Installation options

### Option 1: Clone from Git

```bash
sudo apt-get install git-core
cd ~
git clone https://github.com/ansible/ansible
cd ansible
git log
git submodule update --init --recursive
sudo apt-get install python-jinja2 python-paramiko python-yaml sshpass
```

Activate Ansible's development environment and confirm the binary is available.

```bash
which ansible
less ./hacking/env-setup
source ./hacking/env-setup
which ansible
ansible --version
```

### Option 2: Install from apt

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Prepare target containers with LXC

Install LXC on the control node and create a few Ubuntu containers that mimic a
small environment.

```bash
sudo apt-get update
sudo apt-get install lxc
lxc-ls --fancy
```

Create two web servers and one database server.

```bash
sudo lxc-create -n web1 -t ubuntu
sudo lxc-create -n web2 -t ubuntu
sudo lxc-create -n db1 -t ubuntu
```

List and start the containers as background services.

```bash
lxc-ls -f
sudo lxc-start -n web1 -d
sudo lxc-start -n web2 -d
sudo lxc-start -n db1 -d
```

Log into a container to satisfy Ansible's prerequisites (Python 2.x and SSH).

```bash
sudo lxc-attach -n web1
sudo apt-get install python-minimal
exit
```

Repeat for the remaining containers as needed.

## Create an inventory

Build a simple inventory that groups the LXC instances into logical roles.

```ini
# inventory
[allservers]
10.0.3.113
10.0.3.247
10.0.3.95

[web]
10.0.3.247
10.0.3.95

[database]
10.0.3.113
```

## Run ad-hoc commands

Verify connectivity and perform common administrative actions.

```bash
ansible allservers -m ping -u ubuntu -i inventory
ansible allservers -a "free -h" -u ubuntu -i inventory
ansible allservers -a "apt list --installed | grep nginx" -u ubuntu -i inventory
ansible allservers -a "apt-get update" -u root -i inventory
ansible web -a "apt-get install -y nginx" -u root -i inventory
ansible web -m service -a "name=nginx state=restarted" -u root -i inventory
```
