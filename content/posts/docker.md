+++ 
date = "2020-11-10"
title = "Docker"
slug = "docker" 
tags = []
categories = []
+++

[Best Docker tutorial](https://www.youtube.com/watch?v=zJ6WbK9zFpI&t=1s)

Its purpose is to make a process, with unique environment requirments, run anywhere.

It's built on the Linux kernal. To run on Windows, Docker creates a Linux VM and runs containers in that VM. When the host kernal is Windows or Unix, Docker can interact with it directly.

To run Docker on Mac specifically, the newest and best option is to get Docker Desktop for Mac. Docker Desktop for Mac uses HyperKit (virtualization software) to create a Linux OS. Then Docker runs inside of the Linux OS.

Docker acheives its sandbox effect using namespaces. The host OS Linux has process numbers starting with 1. Each process it opens gets a subsequent number. No two processes can have the same number (hence namespace). So the Docker process actually has a number of say 6 on the host OS. But Docker creates a new namespace for itself and makes its start process number 1. Therefore achieving a kind of sandbox. Everything that dockers needs to do, like create containers, networks, mounts, and etc, use separate namespaces to isolate themselves.

## Container

A process on your machine that is isolated from all other processes.

It uses various layers from its image for its filesystem. However, each container gets a scratch space to add/update/remove files. This scratch space is isolated in each container, even containers built from the same image.

Docker Volumes allow you to connect the filesystem of a container with the host OS filesystem. When directory with data in a container is "mounted," it is connected to the host OS filesystem. __named volumes__ are volumes like buckets of data. named volumes are saved in a docker controlled directory close to root of host OS. __bind mounts__ are another type of volume and way to persist data in docker. With bind mounts you can control the mountpoint on the host.

Containers are not like VMs in that they don't just run on standby, they are not meant to run full operating systems. Containers are environments for running processes. Therefore a contianer only lives as long as the process inside it is alive. When the process finishes, the container exits.

## Dockerfile and Image

A Dockerfile is a set of isntructions for building an image. When an image is built it can then be run. Running an image creates a container. The dockerfile is composed of commands that create image layers. A Dockerfile must be built off a previous image. This is done with the FROM command like this:

```
FROM ubuntu

RUN apt-get update
```

That is why there is a default base image for docker: its Alpine (a lightweight Linux distro). Interestingly, the Dockerfile is not included in the image that made it. The best way to get more information about an image is docker inspect <image_name>.

## Basic Commands

Run the container in the background __-d__ and map port __-p__ 80 of the host to 80 in the container. Without __-d__ you won't get your prompt back and STDOUT of the container will be printed to the terminal.
```sh
docker run -dp 80:80 <image_name>
```

Run a container based off the ubuntu image. If no __ubuntu__ image exists locally, docker will pull it from Docker Hub. Specifically, Docker will look for __uhbuntu/ubuntu:latest__ when __ubuntu__ is passed in. No tag is specified, so docker will use the version of ubuntu tagged with __latest__ on Docker Hub.
```sh
docker run -d ubuntu
```

Run a container based off the verison of ubuntu tagged with __11.2.0__.
```sh
docker run -d ubuntu:11.2.0
```

Run a container in interactive mode __-i__. By default, containers dont listen to STDIN. In interactive mode, they will. __-t__ attaches to the container's terminal. With both of these flags, STDIN and STDOUT will flow through the container.
```sh
docker run -it ubuntu ls /
```

List containers that are running and because __-a__ exited.
```sh
docker ps -a
```

Stop a running container.
```sh
docker stop <container_id>
```

Remove a container.
```sh
docker rm <container_id>
```

Remove all stopped containers.
```sh
docker system prune
```

Execute a command in a running container.
```sh
docker exec <container_id> cat file_in_container
```

## Image

List images.
```sh
docker image list
```

List dangling images. Dangling images are layers that have no relationship to any tagged iamges. They consume disk space, so its usually better to remove them. __-f__ is for filter.
```sh
docker images -f dangling=true
```

Remove an image.
```sh
docker rmi <image_name>
```

Tag an image. 
```sh
docker tag <source_image_name>:<tag> <target_image_name>:<tag>
```

Rename an image.
```sh
docker tag <old_image_name> <new_image_name>
```

Build an iamge off of the Dockerfile in the current directory and tag it `-t` with `image_name`.
```sh
docker build -t <image_name> . 
```

See image layer information.
```sh
docker history <image_name>
```

## Registry

Login to Docker Hub. 
```sh
docker login -u <user_name>
```

Pull image. It will pull from Docker Hub unless a different registry is specified. If you dont specifiy a username, docker will assume the username is the same as the image name. So these two commands will do the same thing:
```sh
docker pull ubuntu

docker pull ubuntu/ubuntu:latest
```

Pull image from a different registry.
```sh
docker pull myregistry.local:5000/testing/test-image
```

Push image. It will push to Docker Hub unless a different registry is specified.
```sh
docker push katharineme/myimage
```

## Volume

Map host data to container to create a __bind-mount__. When the container is running, the data will be accessible by the container at the mapped location. The container will listen to changes made the data and changes made to the data in the container will be saved on the host.
```sh
docker run -v </host/data/dir>:</container/dir> <image_name>
```

Create a __named volume__. named volumes are stored in a directory in the virtual machine docker is running. So they arent accessible in the host CLI.
```sh
docker volume create <volume_name>
```

List named volumes.
```sh
docker volume ls
```

## Container Status

See container details.
```sh
docker inspect <container_name>
```

See container logs (STDOUT).
```sh
docker logs <container_name>
```

## Container Power

Give a container 50% of host CPU
```sh
docker run --cpus=.5 ubuntu
```

Give a container 100Mb of host Memory
```sh
docker run --memory=100m ubuntu
```

## Environment Variables

Run container with environment variable. See a containers current environment variables by running `docker inspect`.
```sh
docker run -e APP_COLOR=blue <image_name>
```

## Dockerfile

Plan Layers
- OS
- source repos
- depedencies with apt
- python dependencies
- copy source to /opt
- run web server command

Write Dockerfile

```
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

COPY . . 

ENTRYPOINT ["sleep"]

WORKDIR /home/work

CMD ["5"]

```

ENTRYPOINT
- when you __docker run__ the container, you can pass in a parameter, which will be added the command you set as the entrypoint. For example if the entrypoint is: __ENTRYPOINT ["sleep"]__ and you run __docker run ubuntu-sleeper 10__, the ubuntu-sleeper container will run the sleep command for 10 seconds, and then exit.

CMD
- the process that is run in the container. When this process is finished, the container exits. If your Dockerfile has __CMD ["sleep", "5"]__ but you run __docker run ubunutu-sleeper sleep 10__ the contianer will run sleep for 10 seconds, then exit, becuase the command you pass in overides the CMD in the Dockerfile.

## docker-compose

docker-compose makes the process of building, running, and linking multiple containers easier. This might be done to create different containers for parts of a single application (one container for the frontend, and one for the sql database). However, docker-compose can also be used to start a single container. docker-compose runs all the containers with their specificed parameters in docker-compose.yml and connects them so they can access each other.

__docker-compose.yml__

```sh
version: 3
services:
    redis:
        image: my_redis_image
    db:
        image: postgres:9.4
    vote:
        image: voting-app
        ports:
            -5000:80
```

Run containers in __docker-compose.yml__
```sh
docker-compose up
```

Exit and remove containers in __docker-compose.yml__ 
```sh
docker-compose down
```

## .dockerignore

While building an image from a Dockerfile, the Docker client will send everything the entire directory the Dockerfile is in to the Docker daemon. So if your project has a lot of data, use __.dockerignore__ to tell the Docker client what not to send to the daemon. This will speed up image building significantly for large projects.