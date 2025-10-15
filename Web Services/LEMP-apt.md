# Install LEMP Stack on Ubuntu with apt

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2017-11-28
- **Tested on:** Ubuntu 16.04

## Install NGINX

```bash
wget http://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
sudo tee /etc/apt/sources.list.d/nginx.list <<'LIST'
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx
deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx
LIST
sudo apt-get update
sudo apt-get install -y nginx
```

(Optional) Build from source:

```bash
sudo apt-get install -y devscripts
sudo apt-get build-dep -y nginx
mkdir -p ~/nginxbuild
cd ~/nginxbuild
apt-get source nginx
```

## Install MySQL

```bash
sudo apt-get install -y mysql-server
sudo mysql_secure_installation
```

## Install PHP-FPM

```bash
sudo apt-get install -y php-fpm php-mysql
sudo sed -i 's/^;cgi.fix_pathinfo=.*/cgi.fix_pathinfo=0/' /etc/php/7.0/fpm/php.ini
sudo systemctl restart php7.0-fpm
```

## Configure NGINX for PHP

```nginx
server {
    listen 80 default_server;
    server_name 192.168.171.131;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        root /var/www/blog2.ducduy.vn/html;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
