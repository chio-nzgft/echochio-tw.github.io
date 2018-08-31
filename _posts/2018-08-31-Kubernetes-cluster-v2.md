---
layout: post
title: CentOS 7 Kubernetes cluster
date: 2018-08-31
tags: docker
---
## Kubernetes cluster on CentOS 7
```
kubemaster: 192.168.1.99
kube2: 192.168.1.109
kube3: 192.168.1.167
```
## SELINUX
```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

Next, disable swap (on all three machines) with the following command:
```
swapoff -a
```

/etc/fstab and comment out the swap entry like this:
```
# /dev/mapper/centos-swap swap swap defaults 0 0
```

enabling the br_netfilter kernel module on all three servers. 

This is done with the following commands:

```
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

It's time to install the necessary Docker tool. On all three machines, 

install the Docker-ce dependencies with the following command:
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

Next, add the Docker-ce repository with the command:
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Install Docker-ce with the command:
```
yum install -y docker-ce
```

vi /etc/yum.repos.d/kubernetes.repo
```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

Save and close that file. Install Kubernetes with the command:
```
yum install -y kubelet kubeadm kubectl
```

Cgroup changes

Now we need to ensure that both Docker-ce and Kubernetes belong to the same control group (cgroup). By default,

Docker should already belong to cgroupfs 

(you can check this with the command docker info | grep -i cgroup). 

To add Kubernetes to this, issue the command:
```
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Restart the systemd daemon and the kubelet service with the commands:
```
systemctl daemon-reload
systemctl restart kubelet
```

Initialize the Kubernetes cluster

adjusting the IP addresses to fit your needs):
```
kubeadm init --apiserver-advertise-address=192.168.1.99 --pod-network-cidr=192.168.1.0/16
```

Once that completes, head over to kube2 and issue the command (adjusting the IP address to fit your needs):
```
kubeadm join 192.168.1.99:6443 --token TOKEN --discovery-token-ca-cert-hash DISCOVERY_TOKEN
```
Where TOKEN and DISCOVERY_TOKEN are the tokens displayed after the initialization command completes.

## Configuring Kubernetes

Before Kubernetes can be used, we must take care of a bit of configuration. 

Issue the following three commands (to create a new .kube configuration directory, 

copy the necessary configuration file, and give the file the proper ownership):
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Deploy flannel network

Now we must deploy the flannel network to the cluster with the command:
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

