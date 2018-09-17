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
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.164
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.165
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.166
```
```
git clone https://github.com/kairen/kubeadm-ansible.git
cd kubeadm-ansible/
```
 hosts.ini
```
[master]
192.168.0.164

[node]
192.168.0.[165:166]

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
192.168.0.164 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.0.165 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.0.166 | SUCCESS => {
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
192.168.0.164              : ok=28   changed=22   unreachable=0    failed=0
192.168.0.165              : ok=20   changed=15   unreachable=0    failed=0
192.168.0.166              : ok=20   changed=15   unreachable=0    failed=0

```

```
[vagrant@master1 ~]$ export KUBECONFIG=/etc/kubernetes/admin.conf
[vagrant@master1 ~]$ kubectl get node
NAME      STATUS    ROLES     AGE       VERSION
master1   Ready     master    27m       v1.11.2
node1     Ready     <none>    26m       v1.11.2
node2     Ready     <none>    26m       v1.11.2
[vagrant@master1 ~]$ kubectl get po -n kube-system
NAME                                   READY     STATUS              RESTARTS   AGE
coredns-78fcdf6894-h8hg7               0/1       ContainerCreating   0          28m
coredns-78fcdf6894-l4dh5               0/1       ContainerCreating   0          28m
etcd-master1                           1/1       Running             0          28m
kube-apiserver-master1                 1/1       Running             0          28m
kube-controller-manager-master1        1/1       Running             0          28m
kube-flannel-ds-6v8cb                  1/1       Running             0          27m
kube-flannel-ds-lx9kj                  1/1       Running             0          27m
kube-flannel-ds-wmmn6                  1/1       Running             0          28m
kube-proxy-7rc6q                       1/1       Running             0          27m
kube-proxy-g7vnk                       1/1       Running             0          27m
kube-proxy-jc9m9                       1/1       Running             0          28m
kube-scheduler-master1                 1/1       Running             0          28m
kubernetes-dashboard-767dc7d4d-xnc9p   0/1       ContainerCreating   0          28m

```

```
[vagrant@master1 ~]$  kubectl apply -f https://k8s.io/docs/tasks/run-application/deployment.yaml
[vagrant@master1 ~]$  kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY     STATUS              RESTARTS   AGE
default       nginx-deployment-67594d6bf6-htd4n      0/1       ContainerCreating   0          34s
default       nginx-deployment-67594d6bf6-r9wds      0/1       ContainerCreating   0          34s
kube-system   coredns-78fcdf6894-h8hg7               0/1       ContainerCreating   0          32m
kube-system   coredns-78fcdf6894-l4dh5               0/1       ContainerCreating   0          32m
kube-system   etcd-master1                           1/1       Running             0          32m
kube-system   kube-apiserver-master1                 1/1       Running             0          32m
kube-system   kube-controller-manager-master1        1/1       Running             0          32m
kube-system   kube-flannel-ds-6v8cb                  1/1       Running             0          31m
kube-system   kube-flannel-ds-lx9kj                  1/1       Running             0          31m
kube-system   kube-flannel-ds-wmmn6                  1/1       Running             0          32m
kube-system   kube-proxy-7rc6q                       1/1       Running             0          31m
kube-system   kube-proxy-g7vnk                       1/1       Running             0          31m
kube-system   kube-proxy-jc9m9                       1/1       Running             0          32m
kube-system   kube-scheduler-master1                 1/1       Running             0          32m
kube-system   kubernetes-dashboard-767dc7d4d-xnc9p   0/1       ContainerCreating   0          32m
[vagrant@master1 ~]$
```

