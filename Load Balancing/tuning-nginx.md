# Tuning NGINX for Caching and Keep-Alive

- **Author:** nduytg
- **Version:** 1.07
- **Date:** 2017-12-20
- **Tested on:** CentOS 7

Optimize NGINX as a reverse proxy with persistent upstream connections and
microcaching.

## Core settings (`nginx.conf`)

```nginx
http {
    keepalive_timeout 120s;
    keepalive_requests 10000;

    upstream backend {
        server 1.2.3.4:8080;
        server 1.2.3.5:8080;
        keepalive 64;
    }

    proxy_cache_path /tmp/nginx_cache levels=1:2 \
        keys_zone=my_cache:10m max_size=2g inactive=60m \
        loader_threshold=300 loader_files=200;

    server {
        listen 80;
        location / {
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Accept-Encoding "";

            proxy_cache my_cache;
            proxy_cache_valid 200 302 60m;
            proxy_cache_valid 404 5m;
            proxy_cache_revalidate on;
            proxy_cache_min_uses 3;
            proxy_cache_lock on;
            proxy_cache_lock_timeout 60s;
            proxy_cache_background_update on;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_ignore_headers Set-Cookie;
            proxy_no_cache $http_x_no_cache;

            proxy_pass http://backend;
        }
    }
}
```

Replace `/tmp/nginx_cache` with a ramdisk such as `/dev/shm/nginx_cache` to store
cache objects in memory.

## Benchmarking

Always benchmark before and after tuning to validate improvements.

```bash
# Randomized siege run (50 concurrent users for 30 seconds)
siege -c50 -d3 -t30s -i blog.ducduy.vn

# ApacheBench
ab -n 1000 -c 100 http://blog.ducduy.vn/

# Quick curl loop
for i in {1..5}; do curl -I http://blog.ducduy.vn; done

# Inspect TCP connections
ss -tpno
```
