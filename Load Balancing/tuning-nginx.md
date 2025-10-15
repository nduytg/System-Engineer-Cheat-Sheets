# Tuning Nginx

- **Author:** nduytg
- **Version:** 1.07
- **Date:** 20/12/17
- **Tested on:** CentOS 7

## Reference
https://www.nginx.com/blog/nginx-caching-guide/
https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching
https://linode.com/docs/web-servers/nginx/configure-nginx-for-optimized-performance/
https://tweaked.io/guide/nginx/
https://www.nginx.com/blog/benefits-of-microcaching-nginx/
https://www.nginx.com/blog/cache-placement-strategies-nginx-plus/

###### Caching + Keep-alive
http {

	keepalive_timeout 120s;
	## Default setting is 100rq/connection
	keepalive_requests 10000;

	upstream {
		server 1.2.3.4:8080;
		server 1.2.3.5:8080;

		keep alive connection to upstream server
		- test: ss -tpno
		keepalive 64;

	}
	...
	## Caching on RAM
	proxy_cache_path /dev/shm/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=200m inactive=60m loader_threshold=300 loader_files=200;
	## Caching on Disk
	proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=2g inactive=60m loader_threshold=300 loader_files=200;

	server {
    ...
		location / {
			proxy_http_version 1.1; # Always upgrade to HTTP/1.1
			proxy_set_header Connection ""; # Enable keepalives
			proxy_set_header Accept-Encoding ""; # Optimize encoding

			proxy_cache my_cache;
			proxy_cache_valid	200 302 60m;
			proxy_cache_valid 404 5m;
			proxy_cache_revalidate on;
			proxy_cache_min_uses 3;

			### Proxy lock
			we allow only 1 req per URI to hit origin
			in case of a cache miss
			proxy_cache_lock on;
			proxy_cache_lock_timeout 60s;
			proxy_cache_background_update on;
			proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;

			### Proxy ignore cookies
			## Ignore Set-cookie header
			proxy_ignore_headers Set-Cookie;
			## Proxy request to upstream if catch X-No-Cache header
			proxy_no_cache $http_x_no_cache;

			### Proxy pass to upstream
			proxy_pass http://my_upstream;
		}
	}

}
###### Benchmark
#### Benchmark your system before and after you have edited the parameters above!!!

#### Benchmark with siege tool
This tells siege to send 50 concurrent users with a random access delay
of 1-3 seconds for 1 minutes.
**i means randomize url selection and -f tells it to read from the file specified.**
siege -c50 -d3 -t30s -i blog.ducduy.vn

## Benchmark with ab
ab -n 1000 -c 100 blog.ducduy.vn/

## Bench mark with curl
for i in {1..5} ; do curl -I blog.ducduy.vn ; done

Show number of connections
ss -tpno
