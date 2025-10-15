# HA Load Balance - keepalived + nginx

- **Author:** nduytg
- **Version:** 0.9
- **Date:** 12/12/17
- **Tested on:** CentOS 7

## Reference
https://www.nginx.com/resources/admin-guide/nginx-ha-keepalived/
http://keepalived.readthedocs.io/en/latest/installing_keepalived.html
http://scale-out-blog.blogspot.com/2011/01/virtual-ip-addresses-and-their.html
http://www.austintek.com/LVS/LVS-HOWTO/HOWTO/LVS-HOWTO.failover.html

#### Option 1: Install Keepalived by yum
yum update
yum install gcc kernel-headers kernel-devel
yum install keepalived

#### Option 2: Install keepalived from source
## Install kernel headers, kernel-devel
yum install gcc kernel-headers kernel-devel

cd ~
mkdir keepalived_source
cd keepalived_source
wget http://keepalived.org/software/keepalived-1.3.9.tar.gz
tar -xvzf keepalived-1.3.9.tar.gz

## Compile and Install
cd keepalived-1.3.9
./configure --with-kernel-dir=/lib/modules/$(uname -r)/build

make -j2 && make install

## Create soft links
- (not yet....)

---

## Enable IP forwarding
- (if need to forward traffic between interfaces)
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf

## Enable binding non-local IP
echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sysctl -p

#### Configure Keepalived service
### Active - Active Mode
### Primary Server
vim /etc/keepalived/keepalived.conf
------------PRIMARY SERVER------------
global_defs {
    vrrp_version 3
}

vrrp_script chk_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VIP1 {
	## Primary private interface
	specify the network interface for the instance to run on
	interface ens34
    	state MASTER
	## Master priority (1-254)
    	priority 200
	## VRRP sending interval
    	advert_int 1

	virtual_router_id 11
	## Primary private IP used to communicate between peers
	unicast_src_ip 192.168.171.128
	unicast_peer {
		192.168.171.129
	}

	virtual_ipaddress
	{
		192.168.31.10/24 dev ens33 label ens33:vip_1
	}

	authentication
	{
		## Use IP-Sec Authentication Header
		## More secure than plain text password
		auth_type AH
		## The auth_pass will only use the first 8 characters entered.
		auth_pass aabbccdd
	}

	track_script
	{
		chk_nginx
	}
}

vrrp_instance VIP2 {
	## Primary private interface
	specify the network interface for the instance to run on
    	interface ens34
    	state BACKUP
	## Backup priority (1-254)
    	priority 100
	## VRRP sending interval
    	advert_int 1

	virtual_router_id 22
	## Primary private IP used to communicate between peers
	unicast_src_ip 192.168.171.128
	## The auth_pass will only use the first 8 characters entered.
	unicast_peer {
		192.168.171.129
	}

	virtual_ipaddress
	{
		192.168.31.20/24 dev ens33 label ens33:vip_2
	}

	authentication
	{
		## Use IP-Sec Authentication Header
		## More secure than plain text password
		auth_type AH
		auth_pass aabbccdd
	}

	track_script
	{
		chk_nginx
	}
}
---

------------SECONDARY SERVER------------
global_defs {
    vrrp_version 3
}

vrrp_script chk_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VIP1 {
	## Secondary private interface
	specify the network interface for the instance to run on
    	interface ens34
    	state BACKUP
	## Master priority (1-254)
    	priority 100
	## VRRP sending interval
    	advert_int 1

	virtual_router_id 11
	## Secondary private IP used to communicate between peers
	unicast_src_ip  192.168.171.129
	unicast_peer {
		192.168.171.128
	}

	virtual_ipaddress
	{
		192.168.31.10 dev ens33 label ens33:vip
	}

	authentication
	{
		## Use IP-Sec Authentication Header
		## More secure than plain text password
		auth_type AH
		## The auth_pass will only use the first 8 characters entered.
		auth_pass aabbccdd
	}

	track_script
	{
		chk_nginx
	}
}

vrrp_instance VIP2 {
	## Secondary private interface
	specify the network interface for the instance to run on
    	interface ens34
   	state MASTER
	## Master priority (1-254)
    	priority 200
	## VRRP sending interval
    advert_int 1

	virtual_router_id 22
	## Secondary private IP used to communicate between peers
	unicast_src_ip  192.168.171.129
	unicast_peer {
		192.168.171.128
	}

	virtual_ipaddress
	{
		192.168.31.20 dev ens33 label ens33:vip
	}

	authentication
	{
		## Use IP-Sec Authentication Header
		## More secure than plain text password
		auth_type AH
		## The auth_pass will only use the first 8 characters entered.
		auth_pass aabbccdd
	}

	track_script
	{
		chk_nginx
	}
}
---

## Start keepalived service on both VM
systemctl start keepalived
systemctl enabled keepalived
systemctl status keepalived

## Check if keepalived works
ip addr show
tail -f /var/log/messages
tcpdump -vvv -n -i ens34 vrrp

## Allow VRRP traffice through iptables
iptables -I INPUT -p vrrp -j ACCEPT
iptables -I OUTPUT -p vrrp -j ACCEPT
