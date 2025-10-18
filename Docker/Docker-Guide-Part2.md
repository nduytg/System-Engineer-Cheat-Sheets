# Docker Guide Part 2: Build and Share Containers

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 2018-01-03
- **Tested on:** CentOS 7

This lab builds a simple Python/Flask container, publishes it to Docker Hub, and
runs it locally.

## Prerequisites

- Docker Engine 1.13 or later (see Part 1)
- Docker Hub account for pushing images

## Create a working directory

```bash
mkdir -p ~/docker-lab
cd ~/docker-lab
```

## Dockerfile

```dockerfile
FROM python:2.7-slim
WORKDIR /app
ADD . /app
RUN pip install --trusted-host pypi.python.org -r requirements.txt
EXPOSE 80
ENV NAME World
CMD ["python", "app.py"]
```

## Application code (`app.py`)

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(
        name=os.getenv("NAME", "world"),
        hostname=socket.gethostname(),
        visits=visits,
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

## Dependencies (`requirements.txt`)

```text
Flask
Redis
```

## Build and run locally

```bash
docker build -t hello1 .
docker images
docker run -d -p 4000:80 hello1
docker container ls
docker container stop <container-id>
```

## Publish to Docker Hub

```bash
docker login
docker tag hello1 nduytg/get-started:part2
docker images
docker push nduytg/get-started:part2
docker run nduytg/get-started:part2
```

Replace `nduytg` with your Docker Hub username when tagging and pushing images.
