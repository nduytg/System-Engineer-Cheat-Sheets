# Install LEMP Stack with yum on CentOS 7

- **Author:** nduytg
- **Version:** 1.1
- **Date:** 2017-11-23
- **Tested on:** CentOS 7

## Install packages

```bash
sudo yum update -y
sudo yum install -y nginx mariadb-server mariadb php php-mysql
```

## Enable services

```bash
sudo systemctl enable --now nginx
sudo systemctl enable --now mariadb
```

Secure MariaDB:

```bash
sudo mysql_secure_installation
```

To automatically restart NGINX after crashes, add `Restart=always` to the
`[Service]` section of `/etc/systemd/system/multi-user.target.wants/nginx.service`
and reload systemd:

```bash
sudo systemctl daemon-reload
sudo systemctl restart nginx
```
