---
layout: post
title: docker container
date: 2018-08-30
tags: docker
---

docker image
docker container
docker tag
docker Repository public
docker Repository private
docker Registry

## 容器（Container）是映像檔（Image）的執行實例（Instance）

## Docker Container 常用的紀錄一下

```
docker pull ubuntu (pull ubuntu image)
docker create -it ubuntu:latest
(create Container)
```

```
-i

Keep STDIN open even if not attached

-t

Allocate a pseudo-TTY
```

```
docker ps -a
(as linux ps , -a as all info)
( or docker ps --all )

docker ps -a --no-trunc 
(will give full command )
```

```
docker start 2c0
( start 2c0 , only input 前三碼就可)
docker stop 2c0
( stop ...)

```

```
docker run ubuntu /bin/echo Hello, Docker!
(will out put)
Hello, Docker!
```

```
docker run -it ubuntu:14.04 /bin/bash
(will into ubuntu container as bash login )
```

```
docker run -d -it ubuntu ping google.com
( -d was Daemonized .....)
docker logs 374
(檢視 ping google.com 指令的輸出  , 374 was CONTAINER ID )
docker logs -f 374
(持續顯示最新的輸出，可以加上 -f 參數 ,  Ctrl + C to stop)
```

```
(背景狀態執行的容器 -d 進入用 exec)
docker exec -ti 374 /bin/bash
(或看 ps 用 ps aux )
docker exec -ti 374 ps aux

(也可用 attach 切換至 container 的終端機 , 不建議除非 container 掛掉無法 bach 進入)
docker attach 374
```

```
docker rm 270
(移除 270.... container)

docker rm -f 270ea7ee8b06
(強制移除 rm)
```

```
docker stop $(docker ps -a -q)
(停止所有 container)
docker rm $(docker ps -a -q)
(移除所有 container)
```
