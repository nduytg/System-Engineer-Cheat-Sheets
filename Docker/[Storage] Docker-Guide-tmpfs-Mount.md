# Docker Guide: `tmpfs` Mounts

- **Author:** nduytg
- **Version:** 0.1
- **Date:** 2018-01-05
- **Tested on:** CentOS 7

`tmpfs` mounts keep data in memory and are ideal for ephemeral state that should
never hit disk.

## Start a container with a `tmpfs` mount

### Using `--mount`

```bash
docker run -d \
  --name tmpfs-example \
  --mount type=tmpfs,destination=/tmp/cache,tmpfs-size=64m \
  busybox sleep 3600
```

### Using `--tmpfs`

```bash
docker run -d \
  --name tmpfs-short \
  --tmpfs /tmp/cache:rw,size=64m \
  busybox sleep 3600
```

## Validate the mount

```bash
docker exec tmpfs-example df -h /tmp/cache
docker exec tmpfs-example sh -c 'echo data > /tmp/cache/example'
docker exec tmpfs-example cat /tmp/cache/example
docker exec tmpfs-example du -sh /tmp/cache
```

Stop and remove the containers when finished:

```bash
docker rm -f tmpfs-example tmpfs-short
```
