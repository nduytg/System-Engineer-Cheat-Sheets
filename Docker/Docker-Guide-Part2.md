# Docker Guide - Part 2: Containers

- **Author:** nduytg
- **Version:** 1.0
- **Date:** 3/1/18
- **Tested on:** CentOS 7

## Reference
https://docs.docker.com/engine/installation/linux/docker-ce/centos/
https://docs.docker.com/get-started/

## Prerequisites
## Install Docker version 1.13 or higher.
## Read the orientation in Part 1.

#### Define a container with Dockerfile
mkdir ~/docker-lab
cd ~/docker-lab

vi Dockerfile
**Content**
## Use an official Python runtime as a parent image
FROM python:2.7-slim

## Set the working directory to /app
WORKDIR /app

## Copy the current directory contents into the container at /app
ADD . /app

## Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

## Make port 80 available to the world outside this container
EXPOSE 80

## Define environment variable
ENV NAME World

## Run app.py when the container launches
CMD ["python", "app.py"]


#### Create the app
vi app.py
**Content**
from flask import Flask
from redis import Redis, RedisError
import os
import socket

## Connect to Redis
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
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)



#### List requirements
vi requirements.txt
**Content**
Flask
Redis



#### Build the app
docker build -t hello1 .
docker images

docker container ls


docker container stop <container-id>


#### Share your image
## Create Docker account on cloud.docker.com
docker login

docker tag hello1 nduytg/get-started:part2

docker images

docker push nduytg/get-started:part2

docker run nduytg/get-started:part2
