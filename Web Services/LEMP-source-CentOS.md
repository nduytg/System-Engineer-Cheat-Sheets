# Build LEMP Stack from Source on CentOS 7

- **Author:** nduytg
- **Version:** 1.1
- **Date:** 2017-11-28
- **Tested on:** CentOS 7 with NGINX 1.13.6

## Preparation

```bash
sudo yum remove -y nginx* mysql* mariadb* php*
sudo rm -rf /etc/nginx /usr/local/php
sudo yum update -y
sudo yum install -y epel-release
sudo yum groupinstall -y "Development Tools"
sudo useradd --system --home /var/cache/nginx --shell /sbin/nologin --comment "nginx user" --user-group nginx
```

## Build NGINX

```bash
sudo yum install -y gcc gcc-c++ make zlib-devel pcre-devel openssl-devel
mkdir -p ~/nginx_source
cd ~/nginx_source
wget http://nginx.org/download/nginx-1.13.6.tar.gz
tar -xvzf nginx-1.13.6.tar.gz
cd nginx-1.13.6
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
make -j"$(nproc)"
sudo make install
sudo cp man/nginx.8 /usr/share/man/man8
```

Create a systemd unit (`/usr/lib/systemd/system/nginx.service`):

```ini
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
```

Enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nginx
```

## Build MySQL (MariaDB compatible)

```bash
sudo yum install -y cmake ncurses-devel
mkdir -p ~/mysql_source
cd ~/mysql_source
wget https://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.37.tar.gz
wget https://codeload.github.com/google/googletest/tar.gz/release-1.6.0
mkdir -p mysql-5.6.37/source_downloads
tar -zxvf mysql-5.6.37.tar.gz
cd mysql-5.6.37
tar -xvzf ../release-1.6.0 -C source_downloads
cmake .
make -j"$(nproc)"
sudo make install
sudo chown -R mysql:mysql /usr/local/mysql
cd /usr/local/mysql
sudo scripts/mysql_install_db --user=mysql --datadir=/var/lib/mysql
sudo cp support-files/mysql.server /etc/init.d/mysqld
sudo chkconfig --add mysqld
sudo chkconfig mysqld on
sudo service mysqld start
sudo mysql_secure_installation
```

For systemd-based autostart, create `/etc/systemd/system/multi-user.target.wants/mysqld.service` with `Restart=always` or rely on `mysqld_safe`.

## Build PHP 7 with FPM

```bash
sudo yum install -y gcc gcc-c++ libxml2-devel pkgconfig openssl-devel \
  bzip2-devel curl-devel libpng-devel libjpeg-devel libXpm-devel \
  freetype-devel gmp-devel libmcrypt-devel mariadb-devel aspell-devel \
  recode-devel autoconf bison re2c libicu-devel
mkdir -p ~/php_source
cd ~/php_source
wget http://sg2.php.net/distributions/php-7.0.6.tar.gz
tar -xvzf php-7.0.6.tar.gz
cd php-7.0.6
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
  --with-zlib \
  --with-gettext \
  --enable-fpm \
  --enable-simplexml \
  --enable-xmlreader \
  --enable-xmlwriter \
  --with-gdbm
make -j"$(nproc)"
sudo make install
sudo cp php.ini-production /usr/local/php/lib/php.ini
sudo cp sapi/fpm/www.conf /usr/local/php/etc/php-fpm.d/www.conf
sudo cp sapi/fpm/php-fpm.conf /usr/local/php/etc/php-fpm.conf
```

Configure the FPM pool (`/usr/local/php/etc/php-fpm.d/www.conf`):

```conf
user = nginx
group = nginx
listen = /var/run/php-fpm.sock
listen.owner = nginx
listen.group = nginx
```

Create a systemd unit (`/usr/lib/systemd/system/php-fpm.service`):

```ini
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
```

Enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now php-fpm
```

## Configure NGINX with PHP-FPM

```nginx
location ~ \.php$ {
    root html;
    fastcgi_pass unix:/var/run/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

## Virtual hosts

```bash
sudo mkdir -p /var/www/{blog,shop,forum}.ducduy.vn/html
sudo chown -R nginx:nginx /var/www
sudo mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled
```

Include the server blocks in `nginx.conf`:

```nginx
http {
    ...
    include /etc/nginx/sites-enabled/*.conf;
    server_names_hash_bucket_size 64;
}
```

Example server block (`/etc/nginx/sites-available/blog.ducduy.vn.conf`):

```nginx
server {
    listen 80;
    server_name blog.ducduy.vn www.blog.ducduy.vn;
    root /var/www/blog.ducduy.vn/html;
    index index.php index.html index.htm info.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/blog.ducduy.vn.conf /etc/nginx/sites-enabled/blog.ducduy.vn.conf
sudo nginx -t
sudo systemctl restart nginx
```
