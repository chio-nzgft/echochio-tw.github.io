---
layout: post
title: Sock5 proxy in docker
date: 2017-04-19
tags: Sock5 proxy docker
---

GIT 在 https://github.com/echochio-tw/sock5-proxy-in-docker

在 docekr 下執行 .....服務就是scok5  8080 port
 
```
docker run --name sock5 -d --restart=always  -p 8080:8080 echochio/sock5
```
