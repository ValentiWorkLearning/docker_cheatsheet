## Docker cheatsheet
Course link: https://www.katacoda.com/courses/docker

### Search an image
``` docker search imagename```

### Running an image
```docker run redis:latest```
### Running an image in background
```docker run -d redis:latest```
### List of running containers
```docker ps```
### Kill an image instance
```docker kill imageName/imageHash```

### Run a container with a forwarded port
```docker run -d --name redisHostPort -p 6379:6379 redis:latest```


### Dynamic port assignment

```shell
docker run -d --name redisDynamic -p 6379 redis:latest
docker port redisDynamic
docker ps
```

### Mapping a directory from the filesystem
```shell
docker run -d -v <host-dir>:<container-dir>

docker run -d --name redisMapped -v /opt/docker/data/redis:data redis:latest
```

## Interactive mode
```sh
#Dockerfile
docker run -it ubuntu bash
```


## Dockerfile example
```docker
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
```shell
docker build -t image_name:version .
docker build -t webserver-image:v1 .
```

## List images
```shell
docker images
```
## Run built image
```shell
## docker run -d -p 80:80 <image-id|friendly-tag-name>
docker run -d -p 80:80 webserver-image:v1
```


## Custom docker images creation
```docker
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html

# Exposing ports EXPOSE 80 433 or EXPOSE 7000-8000

EXPOSE 80

# Run a command RUN <cmdname>


# Launch an application after the container start  ["cmd", "-a", "arga value", "-b", "argb-value"] with CMD or ENTRYPOINT

ENTRYPOINT ["nginx","-g","daemon off;"]

```

## Node JS app
```docker
FROM node:10-alpine
RUN mkdir -p /src/app

WORKDIR /src/app

COPY package.json /src/package.json
RUN npm install

COPY . /src/app

EXPOSE 3000
CMD ["npm", "start"]
```

## OnBuild
```docker
FROM node:7
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
ONBUILD COPY package.json /usr/src/app
ONBUILD RUN npm install
ONBUILD COPY . /usr/src/app
CMD ["npm", "start"]
```
## OnBuild image build
```docker
FROM node:7-onbuild
EXPOSE 3000
```

## Ignore files during the build
```docker
echo passwords.txt >> .dockerignore
```


## Data containers

```shell
docker create -v /config --name dataContainer busybox

## Copy files
docker cp config.conf dataContainer:/config/

## Mount volumes from other containers
docker run --volumes-from dataContainer ubuntu ls /config

## Export/import data container

docker export dataContainer > dataContainer.tar
docker import dataContainer.tar
```

## Network between containers using Links
https://www.katacoda.com/courses/docker/5
```shell

docker run -d --name redis-server redis
docker run --link redis-server:redis alpine env
docker run --link redis-server:redis alpine cat /etc/hosts
docker run --link redis-server:redis alpine ping -c 1 redis
```

## Connect to an application
```shell
docker run -d -p 3000:3000 --link redis-server:redis katacoda/redis-node-docker-example
```

## Connect to CLI
```shell
docker run -it --link redis-server:redis redis redis-cli -h redis
```

## Creating networks between containers using Networks
```shell
docker network create backend-network

##Connect the container to the network
docker run -d --name redis --net=backend-network redis
docker run --net=backend-network alpine cat /etc/resolv.conf

##Connect two containers
docker network create frontend-network
docker network connect frontend-network redis
docker run -d -p 3000:3000 --net=frontend-network katacoda/redis-node-docker-example


## Container aliases
docker network create frontend-network2
docker network connect --alias db frontend-network2 redis

docker run  --net=frontend-network2 alpine ping -c1 db
```

## Network analysis
```shell
docker network ls
docker network inspect frontend-network
docker network disconnect frontend-network redis
```


## Docker volumes
```shell
docker run  -v /docker/redis-data:/data \
  --name r1 -d redis \
  redis-server --appendonly yes

  cat data | docker exec -i r1 redis-cli --pipe
  ls /docker/redis-data
  docker run -v /docker/redis-data:/backup ubuntu ls /backup

## Volumes-from
docker run --volumes-from r1 -it ubuntu ls /data

## Add permissions with :ro mount
docker run -v /docker/redis-data:/data:ro -it ubuntu rm -rf /data
```


## Manage container log files
```shell
## Basic usage(stderr,stdout)
docker logs redis-server


## SysLog
docker run -d --name redis-syslog --log-driver=syslog redis

## Disable all of the logs
docker run -d --name redis-none --log-driver=none redis

## Obtain the selected log driver from the container
docker inspect --format '{{ .HostConfig.LogConfig}}' <container-name>

docker inspect --format '{{ .HostConfig.LogConfig}}' redis-server
```


## Container restart policies
```shell
## Stop on fail
docker run -d --name restart-default scrapbook/docker-restart-example

## Restart three times before stopping
docker run -d --name restart-3 --restart=on-failure:3 scrapbook/docker-restart-example

docker logs restart-3

## Always restart
docker run -d --name restart-always --restart=always scrapbook/docker-restart-example
```

## Docker metadata
```shell

## A simple label
docker run -l user=12345 -d redis

## External labels
echo 'user=123461' >> labels && echo 'role=cache' >> labels

docker run --label-file=labels -d redis
```
### Docker metadata:Dockerfile label
```docker
## Single label
LABEL vendor=Vendorname

## Multiple labels
LABEL vendor=Vendorname \ com.vendor.version=0.0.5 \
    com.vendor.build-date=2020-06-11T10:47:29Z \
    com.vendor.course=Docker
```

## Inspecting labels
```shell

## Query all metadata
docker inspect rd <container-id>

## Just labels from JSON
docker inspect -f "{{json .Config.Labels}}" rd <containerid>

## Inspecting images
docker inspect -f "{{json .ContainerConfig.Labels}}" katacoda-label-example


## Query by label containers
docker ps --filter "label=user=scrapbook"

## Query by label images
docker images --filter "label=vendor=Katacoda"

## Docker daemons labels
docker -d \
    - H unix:///var/run/docker.sock \
    -- label com.katacoda.environment="production" \
    -- label com.katacoda.storage= "ssd"

```

## Formatting PS Output
```shell
## Names and images as a Table
docker ps --format '{{.Names}} container is using {{.Image}} image'
docker ps --format 'table {{.Names}}\t{{.Image}}'
## List IP addresses
docker ps -q | xargs docker inspect --format '{{ .Id }} - {{ .Name}} - {{ .NetworkSettings.IPAddress }}'
```

## Run docker from rootless Users
```shell
## Create Ubuntu user
useradd -m -d /home/lowprivuser -p $(openssl passwd -1 password) lowprivuser
sudo su lopriwuser

## Install rootless docker container
curl -sSL https://get.docker.com/rootless | sh

## Launch Rootless docker daemon
## Exported envvars
export XDG_RUNTIME_DIR=/tmp/docker-1001
export PATH=/home/lowprivuser/bin:$PATH
export DOCKER_HOST=unix:///tmp/docker-1001/docker.sock

mkdir -p ${XDG_RUNTIME_DIR}
/home/lowprivuser/bin/dockerd-rootless.sh --experimental --storage-driver vfs

## Second terminal
sudo su lowprivuser

##Second terminal !export envvars again!
## should works now:
docker ps
docker info
docker runt -it ubuntu bash
id
id; ps aux| grep lowprivuser
```

## Container metrix
```shell
## Single container
docker stats nginx

## Multiple Containers
docker ps -q | xargs docker stats

```

## Multi stage builds
```docker
## File must be called Dockerfile.multi?

## first stage
FROM golang:1.6-alpine
RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsufix cgo -o main .

## second stage
FROM alphine
EXPOSE 80
CMD ["/app"]
COPY --from 0 /app/main /app

## Syntax improvements discussion

## Image building
docker build -f Dockerfile.multi -t golang-app .
docker images

docker run -d -p 80:80 golang-app
curl localhost
```


## Docker with Makefiles
```shell
docker build -t benhall/docker-make-example .
```

```makefile

default: all run

all:
	docker build -t benhall/docker-make-example .

run:
	docker run benhall/docker-make-example
```
