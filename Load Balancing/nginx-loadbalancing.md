# HA Load Balance - keepalived + nginx

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 8/12/17
- **Tested on:** CentOS 7

## Active - Active Model
## VIP with keepalived + Load-Balance/web with nginx

- VIP1: 192.168.31.10
- VIP2: 192.168.31.20
- Master: 192.168.31.130
- Secondary: 192.168.31.131

#### Load Balancer 1
vim /etc/nginx/nginx.conf
**Load Balancer 1**
http {

	upstream backend_sv {
		## Local server
		server 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
		## Neighbor server
		server 192.168.31.131:8080 max_fails=3 fail_timeout=10s;
	}

	map $cookie_backend $sticky_backend {
		backend_cookie1 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
		backend_cookie2 192.168.31.131:8080 max_fails=3 fail_timeout=10s;
		default backend_sv;
	}

	server {
		listen 80;
		location / {
			set $target http://$sticky_backend;
			proxy_pass $target;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwared-Proto $scheme;

			## 50x gateway time-out
			error_page 500 502 503 504 = @backend_down;
		}

		location @backend_down {
			proxy_pass http://backend_sv;
		}

	}
}


**Load Balancer 2**
http {

	upstream backend_sv {
		## Local server
		server 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
		## Neighbor server
		server 192.168.31.130:8080 max_fails=3 fail_timeout=10s;
	}

	map $cookie_backend $sticky_backend {
		backend_cookie1 127.0.0.1:8080;
		backend_cookie2 192.168.31.130:8080;
		default backend_sv;
	}

	server {
		listen 80;
		location / {
			set $target http://$sticky_backend;
			proxy_pass $target;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwared-Proto $scheme;

			## 50x gateway time-out
			error_page 500 502 503 504 = @backend_down;
		}

		location @backend_down {
			proxy_pass http://backend_sv;
		}

	}
}


**Web Backend 1**
server {
    listen 80;
    ...
    location ~ ^/.+\.php(/|$) {
        add_header Set-Cookie "backend=backend_cookie1;Max-Age=3600";
        ...
    }
}


**Web Backend 2**
server {
    listen 80;
    ...
    location ~ ^/.+\.php(/|$) {
        add_header Set-Cookie "backend=backend_cookie2;Max-Age=3600";
        ...
    }
}
