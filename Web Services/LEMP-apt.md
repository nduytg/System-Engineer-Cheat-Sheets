# Install LEMP Stack by Apt on Ubuntu

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 28/11/17
- **Tested on:** Ubuntu 16

- Reference: Nginx Cookbook

#### Install Nginx
## Download Nginx signing key
wget http://nginx.org/keys/nginx_signing.key
apt-key add nginx_signing.key

## Edit /etc/apt/sources.list.d/nginx.list
## Xenial for Ubuntu 16, Trusty for Ubuntu 14
vi /etc/apt/sources.list.d/nginx.list
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx
deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx

## Update and install nginx with Apt-get
apt-get update
apt-get install nginx

#### Custom build (based on mainline installation above)
apt-get install devscripts
apt-get build-dep nginx

mkdir ~/nginxbuild
cd ~/nginxbuild
apt-get source nginx

#### Install MySQL
apt-get install mysql-server
mysql_secure_installation

#### Install PHP for processing
apt-get install php-fpm php-mysql

### Configure PHP Processor
vim /etc/php/7.0/fpm/php.ini
[...]
cgi.fix_pathinfo=0

systemctl restart php7.0-fpm

### Configure NGINX to use PHP Processor
vim /etc/nginx/conf.d/default.conf
---
server {
    listen 80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name 192.168.171.131;

    location / {
        try_files $uri $uri/ =404;
    }

	location ~ \.php$ {
        root           /var/www/blog2.ducduy.vn/html;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }


    location ~ /\.ht {
        deny all;
    }
}
---
