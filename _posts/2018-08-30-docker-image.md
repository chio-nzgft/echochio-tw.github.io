---
layout: post
title: docker image
date: 2018-08-30
tags: docker
---

docker image

docker container

docker tag

docker Repository public

docker Repository private

docker Registry



## docker image 常用的紀錄一下

```
docker search ubuntu
docker search -s 20 ubuntu 
(top 20 ubuntu image)
```

```
docker pull ubuntu 
(拉)
docker pull ubuntu:latest 
(pull with tag(ver) container)

docker images 
( list local images)
(REPOSITORY , TAG ,  IMAGE ID)
```

```
docker inspect ubuntu:14.04 
( list ubuntu 14.04 詳細資訊)
(output with json use less to view)
```

```
docker save -o ubuntu-latest.tar ubuntu:latest 
(ubuntu:latest backup to local as tar file)
```

```
docker load --i ubuntu-latest.tar
(load local file ubuntu-latest.tar to docker container)
```

```
docker rmi ubuntu:14.04 
(rm local image ubuntu:14.04 )
```
