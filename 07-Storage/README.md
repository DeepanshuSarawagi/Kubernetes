# Storage in Kubernetes

## Table of contents:

1. [Storage in Docker](#1-storage-in-docker)

## 1. Storage in Docker:

The default filesystem of docker is /var/lib/docker. We have multiple folders in this FS such as

- aufs
- containers
- image
- volumes

### 1a. Docker's Layered Architecture:

When docker builds images, it builds these in layered architecture. Each line of instruction in Docker file creates a new layer
in the Docker Image with just the changes from previous layer.

```Docker
FROM Ubuntu
RUN apt-get update && apt-get install python -y
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run 
```

Every command in the above Docker file is a layer, since, each layer stores the changes from previous layer, the size of
the image is comparatively smaller. Now let's consider another Docker application.

```Docker2
FROM Ubuntu
RUN apt-get update && apt-get install python -y
RUN pip install flask flas-mysql
COPY app2.py /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app2.py flask run
```

When we build this application, docker will not build first three layers since the first three layers are same between 
Docker and Docker2, only source code is different. This way Docker builds images faster and efficiently saves space.