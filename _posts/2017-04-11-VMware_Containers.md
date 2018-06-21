---
layout: post
title: VMware vSphere® Integrated Containers™ Engine (VIC)
date: 2017-04-11
tags: VMware Esxi vSphere Containers
---
```
參考 https://vmware.github.io/vic-product/assets/files/html/0.8/vic_installation/vch_installer_examples.html#static-ip
```

下載 VIC --> https://bintray.com/vmware/vic/Download ( 我用 vic_0.9.0.tar.gz)

```
Firewall must permit dst 2377/tcp outbound to the VCH management interface
```

PS : 會請你輸入 ESXI 的 root 密碼 

```
# ./vic-machine-linux create --target root@192.168.0.12  --public-network-ip 192.168.0.13/24 --public-network-gateway 192.168.0.2 --no-tlsverify --force --name 'vch' --force
Apr 11 2017 16:42:50.335+08:00 INFO  ### Installing VCH ####
Apr 11 2017 16:42:50.335+08:00 vSphere password for root:
Apr 11 2017 16:42:50.335+08:00 WARN  Using administrative user for VCH operation - use --ops-user to improve security (see -x for advanced help)
Apr 11 2017 16:42:50.335+08:00 INFO  Using public-network-ip as cname where needed - use --tls-cname to override: 192.168.0.13/24
Apr 11 2017 16:42:50.336+08:00 ERROR Provided cname does not match that in existing server certificate:
Apr 11 2017 16:42:50.336+08:00 ERROR Unable to load certificates: cname option doesn't match existing server certificate in certificate path vch
Apr 11 2017 16:42:50.336+08:00 WARN  Ignoring error loading certificates due to --force
Apr 11 2017 16:42:50.336+08:00 INFO  Generating self-signed certificate/key pair - private key in vch/server-key.pem
Apr 11 2017 16:42:50.563+08:00 WARN  Configuring without TLS verify - certificate-based authentication disabled
Apr 11 2017 16:42:50.704+08:00 INFO  Validating supplied configuration
Apr 11 2017 16:42:50.795+08:00 INFO  Using default datastore: datastore1
Apr 11 2017 16:42:50.799+08:00 INFO  Configuring static IP for additional networks using port group "VM Network"
Apr 11 2017 16:42:50.851+08:00 INFO  Firewall status: ENABLED on "/ha-datacenter/host/localhost.tw.demo/localhost.tw.demo"
Apr 11 2017 16:42:50.864+08:00 INFO  Firewall configuration OK on hosts:
Apr 11 2017 16:42:50.864+08:00 INFO     "/ha-datacenter/host/localhost.tw.demo/localhost.tw.demo"
Apr 11 2017 16:42:50.879+08:00 INFO  License check OK
Apr 11 2017 16:42:50.879+08:00 INFO  DRS check SKIPPED - target is standalone host
Apr 11 2017 16:42:50.923+08:00 INFO
Apr 11 2017 16:42:51.000+08:00 INFO  Creating Resource Pool "vch"
Apr 11 2017 16:42:51.005+08:00 INFO  Creating VirtualSwitch
Apr 11 2017 16:42:51.034+08:00 INFO  Creating Portgroup
Apr 11 2017 16:42:51.066+08:00 INFO  Creating appliance on target
Apr 11 2017 16:42:51.071+08:00 INFO  Network role "public" is sharing NIC with "management"
Apr 11 2017 16:42:51.071+08:00 INFO  Network role "client" is sharing NIC with "management"
Apr 11 2017 16:42:53.612+08:00 INFO  Uploading images for container
Apr 11 2017 16:42:53.612+08:00 INFO     "bootstrap.iso"
Apr 11 2017 16:42:53.612+08:00 INFO     "appliance.iso"
Apr 11 2017 16:43:01.062+08:00 INFO  Waiting for IP information
Apr 11 2017 16:43:06.392+08:00 INFO  Waiting for major appliance components to launch
Apr 11 2017 16:43:06.820+08:00 INFO  Obtained IP address for client interface: "192.168.0.13"
Apr 11 2017 16:43:06.820+08:00 INFO  Checking VCH connectivity with vSphere target
Apr 11 2017 16:43:06.899+08:00 INFO  vSphere API Test: https://192.168.0.12 vSphere API target responds as expected
Apr 11 2017 16:43:15.930+08:00 INFO  Initialization of appliance successful
Apr 11 2017 16:43:15.930+08:00 INFO
Apr 11 2017 16:43:15.930+08:00 INFO  VCH Admin Portal:
Apr 11 2017 16:43:15.930+08:00 INFO  https://192.168.0.13:2378
Apr 11 2017 16:43:15.930+08:00 INFO
Apr 11 2017 16:43:15.930+08:00 INFO  Published ports can be reached at:
Apr 11 2017 16:43:15.930+08:00 INFO  192.168.0.13
Apr 11 2017 16:43:15.930+08:00 INFO
Apr 11 2017 16:43:15.930+08:00 INFO  Docker environment variables:
Apr 11 2017 16:43:15.930+08:00 INFO  DOCKER_HOST=192.168.0.13:2376
Apr 11 2017 16:43:15.930+08:00 INFO
Apr 11 2017 16:43:15.930+08:00 INFO  Environment saved in vch/vch.env
Apr 11 2017 16:43:15.930+08:00 INFO
Apr 11 2017 16:43:15.930+08:00 INFO  Connect to docker:
Apr 11 2017 16:43:15.930+08:00 INFO  docker -H 192.168.0.13:2376 --tls info
Apr 11 2017 16:43:15.930+08:00 INFO  Installer completed successfully
```

PS 如果你的網卡是 vic-public ...不是 VM Network ...要加 ...management-network & public-network 參數 如 :

```
./vic-machine-linux create --target root@192.168.0.12 \
--management-network vic-public --public-network vic-public \
--public-network-ip 192.168.0.7/24 --public-network-gateway 192.168.0.2 \
--no-tlsverify --force --name 'vch2' --force
```

查看 ESXi 上的 docker host 是否 OK 

```
#  docker -H 192.168.0.13:2376 --tls info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: v0.8.0-8351-f5d2542
Storage Driver: vSphere Integrated Containers v0.8.0-8351-f5d2542 Backend Engine
VolumeStores:
vSphere Integrated Containers v0.8.0-8351-f5d2542 Backend Engine: RUNNING
 VCH CPU limit: 9259 MHz
 VCH memory limit: 27.88 GiB
 VCH CPU usage: 29 MHz
 VCH memory usage: 2.298 GiB
 VMware Product: VMware ESXi
 VMware OS: vmnix-x86
 VMware OS version: 6.5.0
Plugins:
 Volume:
 Network: bridge
Swarm: inactive
Operating System: vmnix-x86
OSType: vmnix-x86
Architecture: x86_64
CPUs: 9259
Total Memory: 27.88GiB
Name: vch
ID: vSphere Integrated Containers
Docker Root Dir:
Debug Mode (client): false
Debug Mode (server): false
Registry: registry-1.docker.io
Experimental: false
live Restore Enabled: false
```

放個 docker images 在 host 看看

```
# docker -H 192.168.0.13:2376 --tls pull docker.io/echochio/go-web
Using default tag: latest
Pulling from echochio/go-web
a3ed95caeb02: Pull complete
09c979f45514: Pull complete
818a31092953: Pull complete
Digest: sha256:ce364c7be0b2f2833854197f6a26e22b8b37a1254f22277b327d32031b986522
Status: Downloaded newer image for echochio/go-web:latest
# docker -H 192.168.0.13:2376 --tls images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
echochio/go-web     latest              a64d4e6d90c2        2 months ago        140MB
```

執行 docker ...

```
# docker -H 192.168.0.13:2376 --tls  run -d -p 9090:9090 echochio/go-web
a54bad569f6c8430b6945e40a094df294645d9815c33c247af0e438bdc722c30
# docker -H 192.168.0.13:2376 --tls ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                         NAMES
a54bad569f6c        echochio/go-web     "/bin/sh -c /web"   About a minute ago   Up About a minute   192.168.0.13:9090->9090/tcp   suspicious_keller
```

確定 widnows 的 docker 也行 (我的 widnwos  docker  Containers 是 windows 的不是 linux)
```
Microsoft Windows [版本 10.0.14393]
(c) 2016 Microsoft Corporation. 著作權所有，並保留一切權利。

C:\WINDOWS\system32>docker -H 192.168.0.69:2376 --tls  ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                         NAMES
00c18bc471fe        echochio/go-web     "/bin/sh -c /web"   About an hour ago   Up About an hour    192.168.0.13:9090->9090/tcp   web-go

C:\WINDOWS\system32>
```

看到 Esxi 上 多了個 ...... susp ….

<img src="/images/posts/Esxi_VCB/P10.png">

<img src="/images/posts/Esxi_VCB/P11.png">


執行成果是 :

<img src="/images/posts/Esxi_VCB/p12.png">
