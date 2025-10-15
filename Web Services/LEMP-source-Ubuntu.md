# Install LAMP Stack from source on Ubuntu

- **Author:** nduytg
- **Version:** 0.0
- **Date:** 24/11/17
- **Tested on:** Ubuntu 16

- Reference: Nginx Cookbook
https://www.vultr.com/docs/how-to-compile-nginx-from-source-on-ubuntu-16-04
https://shaunfreeman.name/installing-php-7-on-ubuntu-16-04/


##### Remove old install
### Remove nginx
apt list installed nginx
apt remove nginx
rm -rf /etc/nginx
rm -rf /var/www/*

### Remove MySQL
apt list installed mysql-server
or
yum list installed mariadb*

yum remove mysql-server
or
yum remove mariadb*

### Remove PHP
apt list installed php*
apt remove php*clkea

### Remove remaining packages
apt autoremove

##### Install LEMP Stack
#### Preparation
apt update
apt upgrade
apt install build-essential -y

### Download Install Packages
## Nginx 1.13.1 - Mainline version
cd ~
wget https://nginx.org/download/nginx-1.13.1.tar.gz && tar zxvf nginx-1.13.1.tar.gz

## PCRE version 4.4 - 8.40
wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz && tar xzvf pcre-8.40.tar.gz

zlib version 1.1.3 - 1.2.11
wget http://www.zlib.net/zlib-1.2.11.tar.gz && tar xzvf zlib-1.2.11.tar.gz

## OpenSSL version 1.0.2 - 1.1.0
wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz && tar xzvf openssl-1.1.0f.tar.gz

rm -f *.tar.gz
cd ~/nginx-1.13.1
./configure --help

## Compile and Install


make -j2 && make install

### Create file systemd unit file for nginx
vi /usr/lib/systemd/system/nginx.service
## Copy and paste the following content
---
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
---

## Start and enable the Nginx
systemctl start nginx.service && sudo systemctl enable nginx.service

## Check if Nginx will start up after a reboot
systemctl is-enabled nginx.service


#### MySQL (MariaDB)
apt install mysql-server

#### PHP 7
## Install prequesite packages
apt-get install build-essential autoconf bison libxml2-dev libbz2-dev libmcrypt-dev \
    libcurl4-openssl-dev libltdl-dev libpng12-dev libpspell-dev libreadline-dev libicu-dev \
    libxml2-dev libpng-dev libmcrypt-dev libfreetype6 libfreetype6-dev libxslt-dev imagemagick \
    libmagickwand-dev zlib1g-dev libssl-dev libmysqlclient-dev libgdbm-dev libsslcommon2-dev libdb-dev cmake

## Download PHP source
cd ~
mkdir php_source
cd php_source
wget http://sg2.php.net/distributions/php-7.0.6.tar.gz && tar -xvzf php-7.0.6.tar.gz
cd php-7.0.6

## Configure PHP 7
./configure \
	--with-config-file-path=/usr/local/php/etc \
	--with-mysqli=/usr/bin/mysql_config \
	--with-pdo-mysql=/usr/bin/mysql \
	--prefix=/usr/local/php \
	--sbindir=/usr/sbin \
	--bindir=/usr/bin \
	--enable-mbstring \
	--with-curl=/usr/bin/curl \
	--with-bz2 \
	--enable-soap \
	--enable-zip \
	--enable-intl \
	--with-mcrypt=/usr/local/bin/mcrypt \
	--with-xsl \
	--with-openssl \
	--with-gd \
	--with-jpeg-dir \
	--enable-gd-native-ttf \
	--with-freetype-dir \
	--disable-cgi \
	--enable-zip \
	--with-zlib \
	--with-gettext \
	--enable-fpm \
	--enable-simplexml \
	--enable-xmlreader \
	--enable-xmlwriter \
	--with-gdbm

make -j2 && make install

## Configure PHP 7, FPM
mv /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
mv /usr/localphp/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

## Edit FPM pool to run php-fpm
vi /usr/local/php/etc/php-fpm.d/www.conf
[...]
user = nginx
group = nginx
listen = /var/run/php-fpm.sock
listen.owner = nginx
listen.group = nginx


## Auto start PHP-FPM when booting/crashing (Systemd)
vi /usr/lib/systemd/system/php-fpm.service
---
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=simple
PIDFile=/run/php-fpm/php-fpm.pid
ExecStart=/usr/sbin/php-fpm --nodaemonize --fpm-config /usr/local/php/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
---

## Enable and start PHP-FPM (Systemd)
systemctl status php-fpm.service
systemctl start php-fpm.service
systemctl is-enabled php-fpm.service
systemctl enable php-fpm.service
systemctl daemon-reload

#### Integrate PHP-FPM into NGINX
vim /etc/nginx/nginx.conf
---
[...]
location ~ \.php$ {
	root           html;
	fastcgi_pass unix:/var/run/php-fpm.sock;
	fastcgi_index  index.php;
	fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
	include        fastcgi_params;
}
---

#### Create Virtual Hosts
## Create the directory structure
mkdir -p /var/www/blog3.ducduy.vn/html
mkdir -p /var/www/shop3.ducduy.vn/html
mkdir -p /var/www/forum3.ducduy.vn/html

chown -R nginx:nginx /var/www/blog3.ducduy.vn/html
chown -R nginx:nginx /var/www/shop3.ducduy.vn/html
chown -R nginx:nginx /var/www/forum3.ducduy.vn/html

## Create new server block directories
mkdir /etc/nginx/sites-available
mkdir /etc/nginx/sites-enabled

## Edit nginx.conf
## Add these lines to the end of the http{} block
vim /etc/nginx/nginx.conf
---
[...]
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
---

## Create server block file
cp /etc/nginx/conf.d/default.conf /etc/nginx/sites-available/blog3.ducduy.vn.conf

## Edit the new file
vim /etc/nginx/sites-available/blog3.ducduy.vn.conf
---
server {
    listen       80;
    server_name  blog3.ducduy.vn www.blog3.ducduy.vn;

    note that these lines are originally from the "location /" block
    root   /var/www/blog3.ducduy.vn/html;
    index index.php index.html index.htm info.php;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
---

## Enable the new block files
ln -s /etc/nginx/sites-available/blog3.ducduy.vn.conf /etc/nginx/sites-enabled/blog.ducduy.vn.conf
ln -s /etc/nginx/sites-available/forum3.ducduy.vn.conf /etc/nginx/sites-enabled/forum3.ducduy.vn.conf
ln -s /etc/nginx/sites-available/shop3.ducduy.vn.conf /etc/nginx/sites-enabled/shop3.ducduy.vn.conf

nginx -t
systemctl restart nginx
