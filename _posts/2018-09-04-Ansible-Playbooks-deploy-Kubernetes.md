---
layout: post
title: Ansible Playbooks deploy Kubernetes cluster
date: 2018-09-04
tags: docker
---

# All node
```
sudo cat >>/etc/hosts<<EOF
192.168.0.155 k8s-m1
192.168.0.156 k8s-m2
192.168.0.157 k8s-n1
192.168.0.158 k8s-n2
EOF
```

```
yum -y install haproxy keepalived
```
vi /etc/haproxy/haproxy.cfg
```
frontend api *:6443
    default_backend    api

frontend etcd *:2379
    default_backend    etcd

backend api
    balance   leastconn
    server    k8s-m1  k8s-m1:6443  check
    server    k8s-m2  k8s-m2:6443  check  backup

backend etcd
    balance   leastconn
    server    master1  k8s-m1:2379  check
    server    master2  k8s-m2:2379  check  backup
```
k8s-m1

/etc/keepalived/keepalived.conf
```
vrrp_script chk_haproxy {
    script "systemctl is-active haproxy"
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 1
    priority 100
    virtual_ipaddress {
        192.168.0.222
    }
    track_script {
        chk_haproxy
    }
}
```
k8s-m2

/etc/keepalived/keepalived.conf
```
vrrp_script chk_haproxy {
    script "systemctl is-active haproxy"
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 1
    priority 101
    virtual_ipaddress {
        192.168.0.222
    }
    track_script {
        chk_haproxy
    }
}
```
```
systemctl enable keepalived
systemctl start keepalived
```

# k8s-m1 ...
```
sudo yum -y install epel-release
sudo yum -y update
sudo yum -y install ansible python-netaddr git cowsay
```

vi /etc/ansible/ansible.cfg
```
[defaults]
host_key_checking = False
```

```
ssh-keygen -t rsa
```

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:thYCXzkAlx7Rj3ncB2/Pu4yzwJpgo6YdtnpG8RDem+I root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|    ..++         |
|     oo...  .    |
|    o.o.+= . o   |
|     *.oo.+ . +  |
|      * S.   o o |
|     o * o.     o|
|    oo.+o  o    .|
|    oE=.o o ..o. |
|   o*+   o   ooo.|
+----[SHA256]-----+
```

```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k8s-m1
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k8s-m2
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k8s-n1
ssh-copy-id -i ~/.ssh/id_rsa.pub root@k8s-n2
```

```
git clone https://github.com/kairen/kube-ansible.git
cd kube-ansible
```

vi inventory/hosts.ini
```
[etcds]
k8s-m[1:2] ansible_user=root

[masters]
k8s-m[1:2] ansible_user=root

[nodes]
k8s-n1 ansible_user=root
k8s-n2 ansible_user=root

[kube-cluster:children]
masters
nodes
```
vi inventory/group_vars/all.yml
```
kube_version: 1.11.2

container_runtime: containerd

cni_enable: true
container_network: calico
cni_iface: "eth1" # CNI 網路綁定的網卡

vip_interface: "eth1" # VIP 綁定的網卡
vip_address: 192.168.0.222 # VIP 位址

etcd_iface: "eth1" # etcd 綁定的網卡

enable_ingress: true
enable_dashboard: true
enable_logging: true
enable_monitoring: true
enable_metric_server: true

grafana_user: "admin"
grafana_password: "p@ssw0rd"
```
for test hosts

```
ansible -i inventory/hosts.ini all -m ping
```

```
192.168.0.155 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.0.157 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.0.156 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.0.158 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

start install ...
```
ansible-playbook -i inventory/hosts.ini cluster.yml
```

check
```
kubectl get cs
kubectl get no
kubectl get po -n kube-system
```
```
[root@k8s-m1 kube-ansible]# kubectl get po -n kube-system
NAME                             READY     STATUS    RESTARTS   AGE
calico-node-6tbqd                2/2       Running   0          4m
calico-node-b4fml                2/2       Running   0          4m
calico-node-km5km                2/2       Running   0          4m
calico-node-txpqw                2/2       Running   0          4m
coredns-6d98b868c7-zlqjp         1/1       Running   0          5m
kube-apiserver-k8s-m1            1/1       Running   0          5m
kube-apiserver-k8s-m2            1/1       Running   0          5m
kube-controller-manager-k8s-m1   1/1       Running   0          5m
kube-controller-manager-k8s-m2   1/1       Running   0          5m
kube-haproxy-k8s-m1              1/1       Running   0          5m
kube-haproxy-k8s-m2              1/1       Running   0          5m
kube-keepalived-k8s-m1           1/1       Running   0          5m
kube-keepalived-k8s-m2           1/1       Running   0          5m
kube-proxy-d5wpg                 1/1       Running   0          4m
kube-proxy-gl5r6                 1/1       Running   0          5m
kube-proxy-lhl5w                 1/1       Running   0          5m
kube-proxy-mk5bl                 1/1       Running   0          4m
kube-scheduler-k8s-m1            1/1       Running   0          5m
kube-scheduler-k8s-m2            1/1       Running   0          4m
[root@k8s-m1 kube-ansible]# kubectl get no
NAME      STATUS    ROLES     AGE       VERSION
k8s-m1    Ready     master    7m        v1.11.2
k8s-m2    Ready     master    7m        v1.11.2
k8s-n1    Ready     <none>    5m        v1.11.2
k8s-n2    Ready     <none>    5m        v1.11.2
[root@k8s-m1 kube-ansible]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
[root@k8s-m1 kube-ansible]#

```

# Addons dashboard
```
ansible-playbook -i inventory/hosts.ini addons.yml
```
dashboard check
```
kubectl get po,svc -n kube-system -l k8s-app=kubernetes-dashboard
```
check ha
```
export PKI="/etc/kubernetes/pki/etcd"
ETCDCTL_API=3 etcdctl \
    --cacert=${PKI}/etcd-ca.pem \
    --cert=${PKI}/etcd.pem \
    --key=${PKI}/etcd-key.pem \
    --endpoints="https://172.22.132.9:2379" \
    member list
```
start nginx
```
kubectl run nginx --image nginx --restart=Never --port 80
kubectl expose pod nginx --port 80 --type NodePort
kubectl get po,svc
```
check nginx url
```
curl 192.168.0.222:31780
```

rollback install
```
ansible-playbook -i inventory/hosts.ini reset-cluster.yml
```
