---
layout: post
title: Kubernetes cluster(阿里雲)
date: 2018-08-30
tags: Kubernetes
---

Kubernetes cluster

看網路上設定照做

```
lab1: etcd master haproxy keepalived 192.168.105.92
lab2: etcd master haproxy keepalived 192.168.105.93
lab3: etcd master haproxy keepalived 192.168.105.94
lab4: node  192.168.105.95
lab4: node  192.168.105.96

vip(loadblancer ip): 192.168.105.99
```

virtualbox Vagrantfile：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|
    (2..6).each do |i|
      config.vm.define "lab#{i}" do |node|
        node.vm.box = "centos-7.4-docker-17"
        node.ssh.insert_key = false
        node.vm.hostname = "lab#{i}"
        node.vm.network "private_network", ip: "192.168.105.9#{i}"
        node.vm.provision "shell",
          inline: "echo hello from node #{i}"
        node.vm.provider "virtualbox" do |v|
          v.cpus = 2
          v.customize ["modifyvm", :id, "--name", "lab#{i}", "--memory", "2048"]
        end
      end
    end
end
```

備份 yum 來源
```
cd /etc/yum.repos.d
mkdir bak
mv *.repo bak/
```

機器在阿里 改yum 阿里yum (就照抄)
```
vi CentOS-Base.repo
```

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

```
vi epel-7.repo
```

```
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```

```
vi docker-ce.repo
```

```
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```

```
vi kubernetes.repo
```

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

Kubernetes v1.11.1 建議 docker v17.03,v1.11,v1.12,v1.13

這裡看是 v1.13
```
yum -y install docker
systemctl enable docker && systemctl restart docker
```

docker 如果 error
```
Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. Either boot into a newer kernel or disable selinux in docke...-enabled=false)
```
修改/etc/sysconfig/docker中的--selinux-enabled=false


每台機器設定 ....
```
# timezone
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# disable selinux (need reboot)
sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux
setenforce 0

# Docker 1.13 後要 開啟 forward
# disable iptables filter表中FOWARD 會跨 Node 的 Pod 無法通
iptables -P FORWARD ACCEPT

# 要關 swap
# 要拿掉 /etc/fstab 的 swap 那行
swapoff -a

# 要讓內部 cluster 通 關閉防火牆
firewall-cmd --add-rich-rule 'rule family=ipv4 source address=192.168.105.0/24 accept' # # 即时生效方式
firewall-cmd --add-rich-rule 'rule family=ipv4 source address=192.168.105.0/24 accept' --permanent # 永久生效方式

# k8s 設定(照設不然會出錯)
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
sysctl --system

# 加载ipvs
# 如果重開，需要重新加载，寫入 rc.ocal ....
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
lsmod | grep ip_vs

# 每台都加入 hosts 對照表
cat >>/etc/hosts<<EOF
192.168.105.92 lab1
192.168.105.93 lab2
192.168.105.94 lab3
192.168.105.95 lab4
192.168.105.96 lab5
EOF
```

設定 haproxy 與 keepalived
lab1,lab2,lab3 這三台要設定
```
docker pull haproxy:1.7.8-alpine
mkdir /etc/haproxy
cat >/etc/haproxy/haproxy.cfg<<EOF
global
  log 127.0.0.1 local0 err
  maxconn 50000
  uid 99
  gid 99
  #daemon
  nbproc 1
  pidfile haproxy.pid

defaults
  mode http
  log 127.0.0.1 local0 err
  maxconn 50000
  retries 3
  timeout connect 5s
  timeout client 30s
  timeout server 30s
  timeout check 2s

listen admin_stats
  mode http
  bind 0.0.0.0:1080
  log 127.0.0.1 local0 err
  stats refresh 30s
  stats uri     /haproxy-status
  stats realm   Haproxy\ Statistics
  stats auth    will:will
  stats hide-version
  stats admin if TRUE

frontend k8s-https
  bind 0.0.0.0:8443
  mode tcp
  #maxconn 50000
  default_backend k8s-https

backend k8s-https
  mode tcp
  balance roundrobin
  server lab1 192.168.105.92:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
  server lab2 192.168.105.93:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
  server lab3 192.168.105.94:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
EOF

# start haproxy
docker run -d --name my-haproxy \
-v /etc/haproxy:/usr/local/etc/haproxy:ro \
-p 8443:8443 \
-p 1080:1080 \
--restart always \
haproxy:1.7.8-alpine

# 看 log
docker logs my-haproxy

# 可用瀏覽器看狀態
http://192.168.105.92:1080/haproxy-status
http://192.168.105.93:1080/haproxy-status
http://192.168.105.94:1080/haproxy-status

# 拉 keepalived images
docker pull osixia/keepalived:1.4.4

# 起動要有 ip_vs 模組
lsmod | grep ip_vs
modprobe ip_vs

# 啟動 keepalived
# ens32 為 192.168.105.0/24 網段的網卡
docker run --net=host --cap-add=NET_ADMIN \
-e KEEPALIVED_INTERFACE=ens32 \
-e KEEPALIVED_VIRTUAL_IPS="#PYTHON2BASH:['192.168.105.99']" \
-e KEEPALIVED_UNICAST_PEERS="#PYTHON2BASH:['192.168.105.92','192.168.105.93','192.168.105.94']" \
-e KEEPALIVED_PASSWORD=hello \
--name k8s-keepalived \
--restart always \
-d osixia/keepalived:1.4.4

# 看 log
# 會有兩個 backup 一個 master
docker logs k8s-keepalived

# ping test
ping -c4 192.168.105.99

# 如果不成功要砍掉重練 eepalived
#docker rm -f k8s-keepalived
#ip a del 192.168.105.99/32 dev ens32
```

配置與啟動 kubelet
(需全部機器要設定)
```
DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
echo $DOCKER_CGROUPS
cat >/etc/sysconfig/kubelet<<EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
EOF

# 啟動
systemctl daemon-reload
systemctl enable kubelet && systemctl restart kubelet
```

設定第一個 master (在 lab1設)
```
cd /etc/kubernetes
# 配置文件
CP0_IP="192.168.105.92"
CP0_HOSTNAME="lab1"

cat >kubeadm-master.config<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.1
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

apiServerCertSANs:
- "lab1"
- "lab2"
- "lab3"
- "192.168.105.92"
- "192.168.105.93"
- "192.168.105.94"
- "192.168.105.99"
- "127.0.0.1"

api:
  advertiseAddress: $CP0_IP
  controlPlaneEndpoint: 192.168.105.99:8443

etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP0_IP:2379"
      advertise-client-urls: "https://$CP0_IP:2379"
      listen-peer-urls: "https://$CP0_IP:2380"
      initial-advertise-peer-urls: "https://$CP0_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380"
    serverCertSANs:
      - $CP0_HOSTNAME
      - $CP0_IP
    peerCertSANs:
      - $CP0_HOSTNAME
      - $CP0_IP

controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s

networking:
  podSubnet: 10.244.0.0/16

kubeProxy:
  config:
    mode: ipvs
    # mode: iptables
EOF

# pull image
kubeadm config images pull --config kubeadm-master.config

# kubeadm init with config
kubeadm init --config kubeadm-master.config

# if error .... reset 
#kubeadm reset

# ca 傳到 lab2 , lab3 shell (USER=root)

CONTROL_PLANE_IPS="lab2 lab3"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:/etc/kubernetes/pki/ca.crt
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:/etc/kubernetes/pki/ca.key
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:/etc/kubernetes/pki/sa.key
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:/etc/kubernetes/pki/sa.pub
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:/etc/kubernetes/pki/front-proxy-ca.crt
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:/etc/kubernetes/pki/front-proxy-ca.key
    ssh "${USER}"@$host "mkdir -p /etc/kubernetes/pki/etcd"
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:/etc/kubernetes/pki/etcd/ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:/etc/kubernetes/pki/etcd/ca.key
    scp /etc/kubernetes/admin.conf "${USER}"@$host:/etc/kubernetes/admin.conf
done
```

如果 kubeadm init 失敗
要將 image tag 改成官方的image tag 就OK了
```
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver-amd64:v1.11.1 k8s.gcr.io/kube-apiserver-amd64:v1.11.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy-amd64:v1.11.1 k8s.gcr.io/kube-proxy-amd64:v1.11.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd-amd64:3.2.18 k8s.gcr.io/etcd-amd64:3.2.18
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler-amd64:v1.11.1 k8s.gcr.io/kube-scheduler-amd64:v1.11.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager-amd64:v1.11.1 k8s.gcr.io/kube-controller-manager-amd64:v1.11.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.1.3 k8s.gcr.io/coredns:1.1.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
```

設定第二個 master (在 lab2設)
```
cd /etc/kubernetes
# 配置文件
CP0_IP="192.168.105.92"
CP0_HOSTNAME="lab1"
CP1_IP="192.168.105.93"
CP1_HOSTNAME="lab2"

cat >kubeadm-master.config<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.1
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

apiServerCertSANs:
- "lab1"
- "lab2"
- "lab3"
- "192.168.105.92"
- "192.168.105.93"
- "192.168.105.94"
- "192.168.105.99"
- "127.0.0.1"

api:
  advertiseAddress: $CP1_IP
  controlPlaneEndpoint: 192.168.105.99:8443

etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP1_IP:2379"
      advertise-client-urls: "https://$CP1_IP:2379"
      listen-peer-urls: "https://$CP1_IP:2380"
      initial-advertise-peer-urls: "https://$CP1_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380,$CP1_HOSTNAME=https://$CP1_IP:2380"
      initial-cluster-state: existing
    serverCertSANs:
      - $CP1_HOSTNAME
      - $CP1_IP
    peerCertSANs:
      - $CP1_HOSTNAME
      - $CP1_IP

controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s

networking:
  podSubnet: 10.244.0.0/16

kubeProxy:
  config:
    mode: ipvs
    # mode: iptables
EOF

# 配置kubelet
kubeadm alpha phase certs all --config kubeadm-master.config
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-master.config
kubeadm alpha phase kubelet write-env-file --config kubeadm-master.config
kubeadm alpha phase kubeconfig kubelet --config kubeadm-master.config
systemctl restart kubelet

# 加etcd到 cluster
CP0_IP="192.168.105.92"
CP0_HOSTNAME="lab1"
CP1_IP="192.168.105.93"
CP1_HOSTNAME="lab2"
export KUBECONFIG=/etc/kubernetes/admin.conf 
kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP1_HOSTNAME} https://${CP1_IP}:2380
kubeadm alpha phase etcd local --config kubeadm-master.config

# pull image
kubeadm config images pull --config kubeadm-master.config

# 部署
kubeadm alpha phase kubeconfig all --config kubeadm-master.config
kubeadm alpha phase controlplane all --config kubeadm-master.config
kubeadm alpha phase mark-master --config kubeadm-master.config
```

設定第三個 master (在 lab3設)
```
cd /etc/kubernetes
# 配置文件
CP0_IP="192.168.105.92"
CP0_HOSTNAME="lab1"
CP1_IP="192.168.105.93"
CP1_HOSTNAME="lab2"
CP2_IP="192.168.105.94"
CP2_HOSTNAME="lab3"
cat >kubeadm-master.config<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.1
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

apiServerCertSANs:
- "lab1"
- "lab2"
- "lab3"
- "192.168.105.92"
- "192.168.105.93"
- "192.168.105.94"
- "192.168.105.99"
- "127.0.0.1"

api:
  advertiseAddress: $CP2_IP
  controlPlaneEndpoint: 192.168.105.99:8443

etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP2_IP:2379"
      advertise-client-urls: "https://$CP2_IP:2379"
      listen-peer-urls: "https://$CP2_IP:2380"
      initial-advertise-peer-urls: "https://$CP2_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380,$CP1_HOSTNAME=https://$CP1_IP:2380,$CP2_HOSTNAME=https://$CP2_IP:2380"
      initial-cluster-state: existing
    serverCertSANs:
      - $CP2_HOSTNAME
      - $CP2_IP
    peerCertSANs:
      - $CP2_HOSTNAME
      - $CP2_IP

controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s

networking:
  podSubnet: 10.244.0.0/16

kubeProxy:
  config:
    mode: ipvs
    # mode: iptables
EOF

# 配置kubelet
kubeadm alpha phase certs all --config kubeadm-master.config
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-master.config
kubeadm alpha phase kubelet write-env-file --config kubeadm-master.config
kubeadm alpha phase kubeconfig kubelet --config kubeadm-master.config
systemctl restart kubelet

# 加etcd到 cluster
CP0_IP="192.168.105.92"
CP0_HOSTNAME="lab1"
CP2_IP="192.168.105.94"
CP2_HOSTNAME="lab3"
KUBECONFIG=/etc/kubernetes/admin.conf
kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP2_HOSTNAME} https://${CP2_IP}:2380
kubeadm alpha phase etcd local --config kubeadm-master.config

# pull image
kubeadm config images pull --config kubeadm-master.config

# 部署
kubeadm alpha phase kubeconfig all --config kubeadm-master.config
kubeadm alpha phase controlplane all --config kubeadm-master.config
kubeadm alpha phase mark-master --config kubeadm-master.config
```

配置使用kubectl (在其中一台master 例如 lab1)
```
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# 查看node
```
# 查看node節點
kubectl get nodes

# 只有網路安装配置完成，才為 ready 
# 設 master 允許部署，參與工作負載，現在可以部署其他系統組件
# 如 dashboard, heapster, efk等
kubectl taint nodes --all node-role.kubernetes.io/master-
```

網路安装配置(在其中一台master 例如 lab1)
```
cd /etc/kubernetes
mkdir flannel && cd flannel
wget https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml

#修改配置
#此處的ip配置要與上面kubeadm的pod-network一致
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }

# 修改鏡像
image: registry.cn-shanghai.aliyuncs.com/gcr-k8s/flannel:v0.10.0-amd64

# 如果Node有多網卡的话，参考 flannel issues 39701，
# https://github.com/kubernetes/kubernetes/issues/39701
# 目前需要在KUBE-flannel.yml中使用--iface 參數指定cluster 內網網卡的名稱，
# 否則可能會出現的dns無法解析。無法通信的情況，需要將KUBE-flannel.yml下載到本地，
# flanneld啟動參數加上--iface = <IFACE名稱>
    containers:
      - name: kube-flannel
        image: registry.cn-shanghai.aliyuncs.com/gcr-k8s/flannel:v0.10.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=ens32

# 啟動
kubectl apply -f kube-flannel.yml

# 查看
kubectl get pods --namespace kube-system
kubectl get svc --namespace kube-system
```

所有node 加入cluster(在所有機器做 ..lab1 - lab5)
```
# 此命令為初始化master 成功後返回的結果
kubeadm join 192.168.105.99:8443 --token j6zjtl.tgptijigkhhnuc23 --discovery-token-ca-cert-hash sha256:f3e9ae0841084185649b6c111b7e992465b81f2442d42871c6a15731a17dabba
```

如node 報錯處：
```
tail -f /var/log/message

Jul 26 07:52:21 localhost kubelet: E0726 07:52:21.336281   10018 summary.go:102] Failed to get system container stats for "/system.slice/kubelet.service": failed to get cgroup stats for "/system.slice/kubelet.service": failed to get container info for "/system.slice/kubelet.service": unknown container "/system.slice/kubelet.service"


在kubelet配置文件追加以下配置
/etc/sysconfig/kubelet

# Append configuration in Kubelet
--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

安裝儀表板插件 (在其中一台master 例如 lab1)
```
cd /etc/kubernetes
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml 
```

vi kubernetes-dashboard.yaml
```
      - name: kubernetes-dashboard
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.8.3
```


# 安裝Dashboard

```
kubectl create -f kubernetes-dashboard.yaml
kubectl get svc,pod --all-namespaces | grep dashboard
# 可以看到kubernetes的 Dashboard 已正常運行。

kube-system   service/kubernetes-dashboard   NodePort    10.108.96.71   <none>        443:30356/TCP   1m
kube-system   pod/kubernetes-dashboard-754f4d5f69-nfvrk   0/1       CrashLoopBackOff   3          1m
```


Dashboard cluster manager 權限

需要一個admin cluster manager 的權限，

新建 kubernetes-dashboard-admin.rbac.yaml文件，內容如下
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
# Create ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

執行命令
```
kubectl create -f kubernetes-dashboard-admin.rbac.yaml
```

找到kubernete-dashboard-admin的token，用戶登錄使用

執行命令並查看結果
```
[root@lab1 kubernetes]# kubectl -n kube-system get secret | grep admin-user
admin-user-token-b9mpt                           kubernetes.io/service-account-token   3         29s
```

可以看到名稱是kubernetes-dashboard-admin-token-ddskx，使用該名稱執行如下命令
```
[root@lab1 kubernetes]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-b9mpt
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=f1247ca8-9173-11e8-bbc3-000c29ea3e30

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWI5bXB0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmMTI0N2NhOC05MTczLTExZTgtYmJjMy0wMDBjMjllYTNlMzAiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.g9F6vn84ds6iVi1TJViWaK1oHMsY0vQoV5Xq8n0nF5WF4uJkkToHxs0nHO4G4u927ZWsMuF0JiTD4rKB0uJcnHdc4GwBN6L2XMceD69YCpunLlgoFOFbu6z9IsZfyvHvFAYbm0Tv4JWBYxkCqmeTKtL1GOtobIs24dvfXk6inn51ZhTAUW_urWvIn8yqckOBJkq7B_wf6EZA0QeNEhhbt_GHPpwq0CMhk4cWHXh_a27y-qKkpu5Cbo_Ux2kUA44o1wmiHgbw4Lh-__KJY3LZmIu9PZzyWyIPaUWlSQ76GXHyOZQ8dw3WRANi9zaEpiwi4e4XXXXXXXXXXXXXXXXXXXXXXX
ca.crt:     1025 bytes
namespace:  11 bytes
```

記下這 token，等下登錄使用，這個token 是永久的

