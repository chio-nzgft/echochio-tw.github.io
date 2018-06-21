---
layout: post
title: vSphere Docker Volume Service
date: 2017-04-12
tags: Esxi docker
---

先看軟體需求 

Docker Host (VM) running Docker 1.13/17.03 and above

vSphere ESXi 6.0+

vSphere Storage (VSAN, VMFS or NFS) for ESXi host

下載  vib 檔 https://bintray.com/vmware/vDVS/VIB/ 後在 ESXi 上安裝

```
# esxcli software vib install -v /tmp/VMWare_bootbank_esx-vmdkops-service_0.13.15d313a-0.0.1.vib --no-sig-check
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: VMWare_bootbank_esx-vmdkops-service_0.13.15d313a-0.0.1.vib
   VIBs Removed:
   VIBs Skipped:
```

發現 VIC (vSphere Integrated Containers) 不支援 plugin ..... XD
```
# docker -H 192.168.0.69:2376 --tls  plugin ls
Error response from daemon: vSphere Integrated Containers does not yet support plugins
```

在 docker server 安裝 plugin
```
# docker plugin install --grant-all-permissions --alias vsphere store/vmware/docker-volume-vsphere
latest: Pulling from store/vmware/docker-volume-vsphere
bc8c14d82abc: Download complete
Digest: sha256:6b81577c0502f537bc8a5ccf3f1599d9f9b7276242d7043d00d04496f312b976
Status: Downloaded newer image for store/vmware/docker-volume-vsphere:latest
Installed plugin store/vmware/docker-volume-vsphere
```

檢查 plugin
```
#  docker plugin ls
ID                  NAME                DESCRIPTION                           ENABLED
b696c8c93709        vsphere:latest      VMWare vSphere Docker Volume plugin   true
```

建立 docker volume 
```
# docker volume create --driver=vsphere --name=MyVolume1
MyVolume1
```

檢查˙建立的 volume 
```
#  docker volume ls
DRIVER              VOLUME NAME
vsphere:latest      MyVolume1@datastore1
# docker volume inspect MyVolume1
[
    {
        "Driver": "vsphere:latest",
        "Labels": {},
        "Mountpoint": "/mnt/vmdk/MyVolume1",
        "Name": "MyVolume1",
        "Options": {},
        "Scope": "global",
        "Status": {
            "access": "read-write",
            "attach-as": "independent_persistent",
            "capacity": {
                "allocated": "13MB",
                "size": "100MB"
            },
            "clone-from": "None",
            "created": "Wed Apr 12 03:14:11 2017",
            "created by VM": "CentOS7",
            "datastore": "datastore1",
            "diskformat": "thin",
            "fstype": "ext4",
            "status": "detached"
        }
    }
]
```

看到 datastore 多了一個 vmdk

<img src="/images/posts/Esxi_VCB/p3.png">
