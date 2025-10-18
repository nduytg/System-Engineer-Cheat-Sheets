# Docker Guide: Bind Mounts

- **Author:** nduytg
- **Version:** 0.1
- **Date:** 2018-01-05
- **Tested on:** CentOS 7

Bind mounts map a directory or file from the host into a container. They are
ideal for development workflows and when the host needs direct control over the
content.

## Start a container with a bind mount

### Using `--mount`

```bash
mkdir -p $(pwd)/site-content
echo "Hello from bind mount" > $(pwd)/site-content/index.html

docker run -d \
  --name devtest \
  --mount type=bind,source="$(pwd)/site-content",target=/usr/share/nginx/html \
  -p 8080:80 nginx
```

### Using `-v`

```bash
docker run -d \
  --name devtest-v \
  -v "$(pwd)/site-content":/usr/share/nginx/html \
  -p 8081:80 nginx
```

## Validate the mount

```bash
curl http://localhost:8080
curl http://localhost:8081
docker exec devtest ls /usr/share/nginx/html
docker exec devtest cat /usr/share/nginx/html/index.html
```

Changes you make to the files under `site-content` on the host appear instantly
inside both containers.
