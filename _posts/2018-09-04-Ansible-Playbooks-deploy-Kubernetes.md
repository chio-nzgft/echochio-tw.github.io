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
