# Build LEMP Stack from Source on Ubuntu 16.04

- **Author:** nduytg
- **Version:** 0.0
- **Date:** 2017-11-24
- **Tested on:** Ubuntu 16.04

## Preparation

```bash
sudo apt remove -y nginx mysql-server php*
sudo rm -rf /etc/nginx /var/www/*
sudo apt autoremove -y
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

## Build NGINX

```bash
cd ~
wget https://nginx.org/download/nginx-1.13.1.tar.gz && tar zxvf nginx-1.13.1.tar.gz
wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz && tar xzvf pcre-8.40.tar.gz
wget http://www.zlib.net/zlib-1.2.11.tar.gz && tar xzvf zlib-1.2.11.tar.gz
wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz && tar xzvf openssl-1.1.0f.tar.gz
rm -f *.tar.gz
cd ~/nginx-1.13.1
./configure --with-pcre=../pcre-8.40 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.0f
make -j"$(nproc)"
sudo make install
```

Create `/lib/systemd/system/nginx.service` (same content as the CentOS example)
and enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nginx
```

## Install MySQL

```bash
sudo apt install -y mysql-server
sudo mysql_secure_installation
```

## Build PHP 7 with FPM

```bash
sudo apt-get install -y build-essential autoconf bison libxml2-dev libbz2-dev \
  libmcrypt-dev libcurl4-openssl-dev libltdl-dev libpng-dev libpspell-dev \
  libreadline-dev libicu-dev libfreetype6-dev libxslt1-dev imagemagick \
  libmagickwand-dev zlib1g-dev libssl-dev libmysqlclient-dev libgdbm-dev \
  libdb-dev cmake
mkdir -p ~/php_source
cd ~/php_source
wget http://sg2.php.net/distributions/php-7.0.6.tar.gz
tar -xvzf php-7.0.6.tar.gz
cd php-7.0.6
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
  --with-zlib \
  --with-gettext \
  --enable-fpm \
  --enable-simplexml \
  --enable-xmlreader \
  --enable-xmlwriter \
  --with-gdbm
make -j"$(nproc)"
sudo make install
sudo mv /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
sudo mv /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```

Update the FPM pool to run as the `www-data` or `nginx` user and create a
systemd service identical to the CentOS example. Enable it with
`sudo systemctl enable --now php-fpm`.

## Integrate PHP-FPM with NGINX

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
sudo mkdir -p /var/www/blog3.ducduy.vn/html
```

Create additional site blocks following the CentOS example, link them from
`sites-enabled`, reload NGINX, and verify the deployment.
