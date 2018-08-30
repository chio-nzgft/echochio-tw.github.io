---
layout: post
title: docker registry server
date: 2018-08-30
tags: docker
---

private registry server

```
 private registry server 1.0 
 (with python) 
 private registry server 2.0 
 (with go)
```

 docker setup docker Container registry server
```
docker run -d -p 5000:5000 --restart=always --name registry -v /mnt/registry:/var/lib/registry registry:2
(registry store in /var/lib/registry ...  to local  /mnt/registry )
```

```
(pull from  docker hub/public registry server)
docker pull ubuntu
(change tag as private registry image)
docker tag ubuntu localhost:5000/ubuntu
(push image push to private registry server)
docker push localhost:5000/ubuntu
 (private registry server pull image)
docker pull localhost:5000/ubuntu
```

```
(login public Docker hub )
docker login -e test@gmail.com -p testpass -u testuser
```
