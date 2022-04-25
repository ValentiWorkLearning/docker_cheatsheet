## Docker cheatsheet

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

## Networka between containers using Links
