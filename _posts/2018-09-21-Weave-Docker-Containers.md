---
layout: post
title: Weave-Docker-Containers
date: 2018-09-21
tags: docker
---

#  Weave Scope In Docker Containers
```
docker run -d --name redis redis
docker run -d --link redis:redis katacoda/redis-node-docker-example
scope launch
```

```
Unable to find image 'weaveworks/scope:1.6.5' locally
1.6.5: Pulling from weaveworks/scope
019300c8a437: Pull complete
c43acb0378da: Pull complete
a80c942267bc: Pull complete
d2fce4cb1694: Pull complete
a6abab25d397: Pull complete
279c4121e3f4: Pull complete
9fa23034dd9f: Pull complete
597eea2280da: Pull complete
69f828530b80: Pull complete
e957c5137988: Pull complete
4bd72719a6d1: Pull complete
Digest: sha256:1725a1b4a2147609c56ae33d11d6b1cd72bb5326a1f9d2ce4d5c0b4c47dffd28
Status: Downloaded newer image for weaveworks/scope:1.6.5
a0b87e9ac69393e250f9a4c24192fe7db3dd9cf59b58e00414c400048b743904
Scope probe started
Weave Scope is listening at the following URL(s) of host host01:
  * http://172.17.0.17:4040/
```


<img src="/images/posts/docker/3.png">


#NOT in docker ...

```
sudo wget -O /usr/local/bin/scope https://github.com/weaveworks/scope/releases/download/latest_release/scope
sudo chmod a+x /usr/local/bin/scope
sudo scope launch
```
