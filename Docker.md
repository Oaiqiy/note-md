---
title: "Docker"
date: 2022-06-03T20:37:06+08:00
draft: false
---

## Commands

`docker version` view version

`docker info` view docker system info

`docker images` view local images `-a` view all images

`docker search [images]` search images

`docker pull [image:tag]` download image

`docker rmi [image ID]` delete image

`docker run [image:tag] [command]` create a container

     -i interactive operation
     -t terminal
     -d run in backend
     -P random map network port
     -p [external port]:[internal port] map port
     --name [container name]

`docker ps` show all running containers

    -l show lasting launched container
    -a show all containers
    -q show lasting launched container's id
  