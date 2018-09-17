---
layout: post
title: Ansible Playbooks deploy Kubernetes site
date: 2018-09-04
tags: docker
---

win7 install Vagrant

下載 Vagrant 安裝重開
下載 virtualbox 安裝重開
更新 powershell 

```
https://docs.microsoft.com/zh-tw/powershell/wmf/5.1/install-configure
```

設定 
E:\vagrant\Vagrantfile
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/centos-7.5"
  # If you run into issues with Ansible complaining about executable permissions,
  # comment the following statement and uncomment the next one.
  config.vm.synced_folder ".", "/vagrant"
  # config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
  end
  config.vm.define :master, primary: true do |master|
    master.vm.network :forwarded_port, host: 8001, guest: 8001
    master.vm.network :forwarded_port, host: 2201, guest: 22, id: "ssh", auto_correct: true
    master.vm.network "private_network", ip: "192.168.22.164"
	master.vm.hostname = "master"
	master.vm.provision "shell", path: "bootstrap.sh"
	master.vm.provision "shell", path: "install_ansible.sh"
	
  end
  config.vm.define :node1 do |node1|
    node1.vm.network :forwarded_port, host: 2202, guest: 22, id: "ssh", auto_correct: true
    node1.vm.network "private_network", ip: "192.168.22.165"
    node1.vm.hostname = "node1"
	node1.vm.provision "shell", path: "bootstrap.sh"
  end
  config.vm.define :node2 do |node2|
    node2.vm.network :forwarded_port, host: 2203, guest: 22, id: "ssh", auto_correct: true
    node2.vm.network "private_network", ip: "192.168.22.166"
    node2.vm.hostname = "node2"
	node2.vm.provision "shell", path: "bootstrap.sh"
  end
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end
# if Vagrant.has_plugin?("vagrant-proxyconf")
#   config.proxy.http     = "http://proxy.company.com:8080/"
#   config.proxy.https    = "http://proxy.company.com:8080/"
#   config.proxy.no_proxy = "localhost,127.0.0.1"
# end
end
```
E:\vagrant\bootstrap.sh
```
cat << 'EOF' >> /etc/hosts
192.168.22.164 master
192.168.22.165 node1
192.168.22.166 node2
EOF
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
yum -y update
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```
E:\vagrant\install_ansible.sh
```
yum -y install epel-release
yum -y update
yum -y install ansible python-netaddr git
```
開始安裝
```
Vagrant up
```

裝完後看狀況
```
vagrant global-status
```

```
id       name   provider   state   directory
-----------------------------------------------------------------------
f6ace63  master  virtualbox running C:/vagrant
5b94637  node1  virtualbox running C:/vagrant
0da4a16  node2  virtualbox running C:/vagrant

```
ssh 到 master

sudo vi /etc/ansible/ansible.cfg
```
[defaults]
host_key_checking = False
```

```
# ssh-keygen -t rsa
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
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.22.164
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.22.165
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.22.166
```
```
git clone https://github.com/kairen/kubeadm-ansible.git
cd kubeadm-ansible/
```
 hosts.ini
```
[master]
192.168.22.164

[node]
192.168.22.[165:166]

[kube-cluster:children]
master
node
```
 group_vars/all.yml
```
# Ansible
# ansible_user: root

# Kubernetes
kube_version: v1.11.1
token: b0f7b8.8d1767876297d85c

# 1.8.x feature: --feature-gates SelfHosting=true
init_opts: ""

# Any other additional opts you want to add..
kubeadm_opts: ""
# For example:
# kubeadm_opts: '--apiserver-cert-extra-sans "k8s.domain.com,kubernetes.domain.com"'

service_cidr: "10.96.0.0/12"
pod_network_cidr: "10.244.0.0/16"

# Network implementation('flannel', 'calico')
network: flannel

# Change this to an appropriate interface, preferably a private network.
# For example, on DigitalOcean, you would use eth1 as that is the default private network interface.
cni_opts: "interface=eth1" # flannel: --iface=eth1, calico: interface=eth1

enable_dashboard: yes

# A list of insecure registrys you might need to define
insecure_registrys: ""
# insecure_registrys: ['gcr.io']

systemd_dir: /lib/systemd/system
system_env_dir: /etc/sysconfig
network_dir: /etc/kubernetes/network
kubeadmin_config: /etc/kubernetes/admin.conf
kube_addon_dir: /etc/kubernetes/addon
```
ping test
```
ansible -i hosts.ini all -m ping
```
```
192.168.22.164 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.22.165 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.22.166 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

開始安裝
```
ansible-playbook site.yaml
```

```
PLAY RECAP ******************************************************************************************
192.168.22.164              : ok=28   changed=22   unreachable=0    failed=0
192.168.22.165              : ok=20   changed=15   unreachable=0    failed=0
192.168.22.166              : ok=20   changed=15   unreachable=0    failed=0

```

```
[root@master kubeadm-ansible]# export KUBECONFIG=/etc/kubernetes/admin.conf
[root@master kubeadm-ansible]# kubectl get pods  -n kube-system
NAME                                      READY     STATUS    RESTARTS   AGE
calico-etcd-7qqn7                         1/1       Running   0          3m
calico-kube-controllers-c4df9646f-cwb7m   1/1       Running   0          3m
calico-node-flb9m                         2/2       Running   1          1m
calico-node-ksq49                         2/2       Running   1          3m
calico-node-zhqb5                         2/2       Running   0          1m
coredns-78fcdf6894-d4znm                  1/1       Running   0          3m
coredns-78fcdf6894-jk6l6                  1/1       Running   0          3m
etcd-master                               1/1       Running   0          3m
kube-apiserver-master                     1/1       Running   0          3m
kube-controller-manager-master            1/1       Running   0          3m
kube-proxy-4dphn                          1/1       Running   0          3m
kube-proxy-8525v                          1/1       Running   0          1m
kube-proxy-j54ct                          1/1       Running   0          1m
kube-scheduler-master                     1/1       Running   0          3m
kubernetes-dashboard-767dc7d4d-t65gj      1/1       Running   0          3m
```

```
[root@master kubeadm-ansible]# kubectl -n kube-system get service kubernetes-dashboard
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes-dashboard   ClusterIP   10.101.53.84   <none>        443/TCP   15m
```

