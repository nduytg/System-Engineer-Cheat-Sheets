# NGINX Load Balancing with Sticky Sessions

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2017-12-08
- **Tested on:** CentOS 7

Deploy an active-active pair of NGINX load balancers fronted by Keepalived
virtual IPs. Each node proxies traffic to its local backend and the peer node
while honoring affinity based on a cookie.

## Topology

- VIP1: `192.168.31.10`
- VIP2: `192.168.31.20`
- Load balancer 1: `192.168.31.130`
- Load balancer 2: `192.168.31.131`

## Load balancer 1 (`/etc/nginx/nginx.conf`)

```nginx
http {
    upstream backend_sv {
        server 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
        server 192.168.31.131:8080 max_fails=3 fail_timeout=10s;
    }

    map $cookie_backend $sticky_backend {
        backend_cookie1 127.0.0.1:8080;
        backend_cookie2 192.168.31.131:8080;
        default backend_sv;
    }

    server {
        listen 80;
        location / {
            set $target http://$sticky_backend;
            proxy_pass $target;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
            error_page 500 502 503 504 = @backend_down;
        }

        location @backend_down {
            proxy_pass http://backend_sv;
        }
    }
}
```

## Load balancer 2 (`/etc/nginx/nginx.conf`)

```nginx
http {
    upstream backend_sv {
        server 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
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
            proxy_set_header X-Forwarded-Proto $scheme;
            error_page 500 502 503 504 = @backend_down;
        }

        location @backend_down {
            proxy_pass http://backend_sv;
        }
    }
}
```

## Backend servers

Each PHP backend sets a cookie to keep users pinned to the same node.

```nginx
server {
    listen 80;
    # ... other configuration ...
    location ~ ^/.+\.php(/|$) {
        add_header Set-Cookie "backend=backend_cookie1;Max-Age=3600";
        # PHP-FPM proxy configuration
    }
}
```

On the second backend, set `backend_cookie2` in the header. Adjust cookie names
and upstream weights to reflect your environment.
