# Install LEMP Stack from source on CentOS

- **Author:** nduytg
- **Version:** 1.1
- **Date:** 28/11/17

###### Tested on
CentOS 7
Nginx 1.13.6


## Reference
https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/
https://www.digitalocean.com/community/tutorials/how-to-compile-nginx-from-source-on-a-centos-6-4-x64-vps
http://nginx.org/en/docs/configure.html
https://www.vultr.com/docs/how-to-compile-nginx-from-source-on-centos-7
https://downloads.mariadb.org/mariadb/repositories/#mirror=Beritagar&distro=CentOS&distro_release=centos7-amd64--centos7&version=10.1
https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples
https://shaunfreeman.name/compiling-php-7-on-centos/
https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-centos-7

###### Remove old install
### Remove nginx
yum list installed nginx*
yum remove nginx*
rm -rf /etc/nginx

### Remove MySQL
yum list installed mysql*
or
yum list installed mariadb*

yum remove mysql*
or
yum remove mariadb*

### Remove PHP
yum list installed php*
yum remove php*
rm -rf /usr/local/php/*

##### Install LEMP Stack
#### Preparation
yum update
yum install -y epel-release
yum groupinstall "Development Tools"

## Add Nginx user and group
useradd --system --home /var/cache/nginx --shell /sbin/nologin --comment "nginx user" --user-group nginx

#### Nginx
## Install prerequisite packages

yum -y install gcc gcc-c++ make zlib-devel pcre-devel openssl-devel

cd ~
mkdir nginx_source
cd nginx_source
wget http://nginx.org/download/nginx-1.13.6.tar.gz
tar -xvzf nginx-1.13.6.tar.gz

## Configure Nginx
./configure \
	--user=nginx \
	--group=nginx \
	--prefix=/etc/nginx \
	--sbin-path=/usr/sbin/nginx \
	--conf-path=/etc/nginx/nginx.conf \
	--pid-path=/var/run/nginx.pid \
	--lock-path=/var/run/nginx.lock \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--with-http_gzip_static_module \
	--with-http_stub_status_module \
	--with-http_ssl_module \
	--with-pcre \
	--with-file-aio \
	--with-http_realip_module \
	--without-http_scgi_module \
	--without-http_uwsgi_module

## Compile and Install
make && make install

## Check if nginx is good
nginx -V
nginx -t

## Copy Nginx Man page to /usr/share/man/man8
cp ./man/nginx.8 /usr/share/man/man8
man nginx

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

### Configure Auto-start service after rebooting or crashing
vi /etc/systemd/system/multi-user.target.wants/nginx.service
---
[Service]
...
Restart=always
...
---

systemctl status nginx
systemctl daemon-reload
systemctl status nginx

## Optional
## Plage syntax highlighting of NGINX for vim
mkdir ~/.vim/
cp -r ~/nginx-1.13.6/contrib/vim/* ~/.vim/

#### MySQL (MariaDB)
## Instal prerequisite packages
yum install cmake ncurses-devel

## Download source code
cd ~
mkdir mysql_source
cd mysql_source
wget https://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.37.tar.gz
wget https://codeload.github.com/google/googletest/tar.gz/release-1.6.0

tar -zxvf mysql-5.6.37.tar.gz
tar -xvzf release-1.6.0 -d mysql-5.6.37/source_downloads

## Compile and Install
cmake .
## Compile with 2 cpus
make -j2
make install

## Change owner and group of mysql folder
chown -R mysql:mysql /usr/local/mysql

## Install new database
cd /usr/local/mysql
scripts/mysql_install_db --user=mysql --datadir=/var/lib/mysql

## Auto startup on boot
cp support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
service mysqld start

### Auto start when crashing
## For CentOS 6 (Upstart/init.d)
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
chkconfig --add mysql
service mysql restart
## Service will be controlled and restart by mysqld_safe

## For CentOS 7 (Systemd/Systemctl)
## Use systemctl to start, enable service
systemctl start mysqld && systemctl enable mysqld
nano /etc/systemd/system/multi-user.target.wants/mysqld.service

---
[Service]
...
...
Restart=always
...
---

/usr/local/mysql/bin/mysql_secure_installation

#### PHP
## Install prerequisite packages
yum install gcc gcc-c++  \
	libxml2-devel pkgconfig openssl-devel bzip2-devel \
	curl-devel libpng-devel libjpeg-devel libXpm-devel \
	freetype-devel gmp-devel libmcrypt-devel mariadb-devel \
	aspell-devel recode-devel autoconf bison re2c libicu-devel

## Download PHP source
cd ~
mkdir php_source
cd php_source
wget http://sg2.php.net/distributions/php-7.0.6.tar.gz
tar -xvzf php-7.0.6.tar.gz
cd php-7.0.6

## Configure PHP 7
./configure \
	--with-config-file-path=/usr/local/php/etc \
	--with-mysqli=/usr/local/mysql/bin/mysql_config \
	--with-pdo-mysql=/usr/local/mysql/bin/mysql \
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

## Compile and Install
make -j2 && make install

## Configure PHP 7, FPM
sudo cp -v ./php.ini-production /usr/local/php/lib/php.ini
sudo cp -v ./sapi/fpm/www.conf /usr/local/php/etc/php-fpm.d/www.conf
sudo cp -v ./sapi/fpm/php-fpm.conf /usr/local/php/etc/php-fpm.conf


## Edit FPM pool to run php-fpm
vi /usr/local/php/etc/php-fpm.d/www.conf
[...]
user = nginx
group = nginx
listen = /var/run/php-fpm.sock
listen.owner = nginx
listen.group = nginx

## Auto start PHP-FPM when booting/crashing (CentOS 7)
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

## Enable and start PHP-FPM (CentOS 7)
systemctl status php-fpm.service
systemctl start php-fpm.service
systemctl is-enabled php-fpm.service
systemctl enable php-fpm.service

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
mkdir -p /var/www/blog.ducduy.vn/html
mkdir -p /var/www/shop.ducduy.vn/html
mkdir -p /var/www/forum.ducduy.vn/html

chown -R nginx:nginx /var/www/blog.ducduy.vn/html
chown -R nginx:nginx /var/www/shop.ducduy.vn/html
chown -R nginx:nginx /var/www/forum.ducduy.vn/html

## Create new server block directories
mkdir /etc/nginx/sites-available
mkdir /etc/nginx/sites-enabled

## Edit nginx.conf
## Add these lines to the end of the http{} block
---
[...]
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
---

## Create server block file
cp /etc/nginx/conf.d/default.conf /etc/nginx/sites-available/blog.ducduy.vn.conf

## Edit the new file
vim /etc/nginx/sites-available/blog.ducduy.vn.conf
---
server {
    listen       80;
    server_name  blog.ducduy.vn www.blog.ducduy.vn;

    note that these lines are originally from the "location /" block
    root   /var/www/blog.ducduy.vn/html;
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
ln -s /etc/nginx/sites-available/blog.ducduy.vn.conf /etc/nginx/sites-enabled/blog.ducduy.vn.conf
ln -s /etc/nginx/sites-available/blog.ducduy.vn.conf /etc/nginx/sites-enabled/forum.ducduy.vn.conf
ln -s /etc/nginx/sites-available/blog.ducduy.vn.conf /etc/nginx/sites-enabled/shop.ducduy.vn.conf

nginx -t
systemctl restart nginx
