# High Availability with Keepalived and NGINX

- **Author:** nduytg
- **Version:** 0.9
- **Date:** 2017-12-12
- **Tested on:** CentOS 7

Deploy an active-active pair of NGINX servers protected by VRRP virtual IPs
managed by Keepalived.

## Installation

### Option 1: YUM packages

```bash
sudo yum update -y
sudo yum install -y gcc kernel-headers kernel-devel keepalived
```

### Option 2: Build from source

```bash
sudo yum install -y gcc kernel-headers kernel-devel
mkdir -p ~/keepalived_source
cd ~/keepalived_source
wget http://keepalived.org/software/keepalived-1.3.9.tar.gz
tar -xvzf keepalived-1.3.9.tar.gz
cd keepalived-1.3.9
./configure --with-kernel-dir=/lib/modules/"$(uname -r)"/build
make -j2
sudo make install
```

## Kernel tuning

Allow the hosts to bind the floating IPs:

```bash
echo "net.ipv4.ip_nonlocal_bind = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Enable IP forwarding if the servers will route traffic:

```bash
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Keepalived configuration

Both servers share the same structure; priorities determine which host owns each
virtual IP. Adjust interface names (`ens33`, `ens34`) and IP addresses to match
your environment.

### Primary server (`/etc/keepalived/keepalived.conf`)

```conf
global_defs {
    vrrp_version 3
}

vrrp_script chk_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VIP1 {
    interface ens34
    state MASTER
    priority 200
    advert_int 1
    virtual_router_id 11
    unicast_src_ip 192.168.171.128
    unicast_peer {
        192.168.171.129
    }
    virtual_ipaddress {
        192.168.31.10/24 dev ens33 label ens33:vip_1
    }
    authentication {
        auth_type AH
        auth_pass aabbccdd
    }
    track_script {
        chk_nginx
    }
}

vrrp_instance VIP2 {
    interface ens34
    state BACKUP
    priority 100
    advert_int 1
    virtual_router_id 22
    unicast_src_ip 192.168.171.128
    unicast_peer {
        192.168.171.129
    }
    virtual_ipaddress {
        192.168.31.20/24 dev ens33 label ens33:vip_2
    }
    authentication {
        auth_type AH
        auth_pass aabbccdd
    }
    track_script {
        chk_nginx
    }
}
```

### Secondary server (`/etc/keepalived/keepalived.conf`)

```conf
global_defs {
    vrrp_version 3
}

vrrp_script chk_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VIP1 {
    interface ens34
    state BACKUP
    priority 100
    advert_int 1
    virtual_router_id 11
    unicast_src_ip 192.168.171.129
    unicast_peer {
        192.168.171.128
    }
    virtual_ipaddress {
        192.168.31.10/24 dev ens33 label ens33:vip
    }
    authentication {
        auth_type AH
        auth_pass aabbccdd
    }
    track_script {
        chk_nginx
    }
}

vrrp_instance VIP2 {
    interface ens34
    state MASTER
    priority 200
    advert_int 1
    virtual_router_id 22
    unicast_src_ip 192.168.171.129
    unicast_peer {
        192.168.171.128
    }
    virtual_ipaddress {
        192.168.31.20/24 dev ens33 label ens33:vip
    }
    authentication {
        auth_type AH
        auth_pass aabbccdd
    }
    track_script {
        chk_nginx
    }
}
```

## Start Keepalived

```bash
sudo systemctl enable --now keepalived
sudo systemctl status keepalived
```

## Validate failover

```bash
ip addr show
tail -f /var/log/messages
sudo tcpdump -vvv -n -i ens34 vrrp
```

Allow VRRP traffic through the local firewall:

```bash
sudo iptables -I INPUT -p vrrp -j ACCEPT
sudo iptables -I OUTPUT -p vrrp -j ACCEPT
```

Shut down NGINX or Keepalived on one node to observe the VIP failover behavior.
