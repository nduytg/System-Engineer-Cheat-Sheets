# Install LEMP Stack by Yum on CentOS

- **Author:** nduytg
- **Version:** 1.1
- **Date:** 23/11/17
- **Tested on:** CentOS 7

yum update

#### Install LEMP Stack
## Apache
yum install nginx
systemctl start nginx
systemctl enable nginx

## PHP
yum install php php-mysql


## MySQL (MariaDB)
yum install mariadb-server mariadb
mysql_secure_installation
systemctl start mariadb
systemctl enable mariadb

#### Configure Auto-start service after rebooting or crashing
systemctl is-enabled nginx
systemctl status nginx
systemctl enable nginx
systemctl is-enabled nginx

vi /etc/systemd/system/multi-user.target.wants/nginx.service
[Service]
...
...
Restart=always
...

systemctl status nginx
=> Get PID
### Reload daemon
systemctl daemon-reload
kill -9 PID
systemctl status nginx
=> Reboot after being killed
