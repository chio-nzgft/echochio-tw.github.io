win7 install Vagrant

下載 Vagrant 安裝重開
下載 virtualbox 安裝重開
更新 powershell 

```
https://docs.microsoft.com/zh-tw/powershell/wmf/5.1/install-configure
```

設定 
E:\vagrant\kube1\Vagrantfile
```
Vagrant.configure("2") do |kube1|
  kube1.vm.box = "bento/centos-7.5"
  kube1.vm.define "kube1"
  kube1.vm.network "public_network", use_dhcp_assigned_default_route: true
	kube1.vm.provider "virtualbox" do |v|
		v.memory = 2048
		v.cpus = 4
	end
end
```
E:\vagrant\kube2\Vagrantfile
```
Vagrant.configure("2") do |kube2|
  kube2.vm.box = "bento/centos-7.5"
  kube2.vm.define "kube2"
  kube2.vm.network "public_network", use_dhcp_assigned_default_route: true
    kube2.vm.provider "virtualbox" do |v|
		v.memory = 2048
		v.cpus = 4
	end
end
```
E:\vagrant\kube3\Vagrantfile
```
Vagrant.configure("2") do |kube3|
  kube3.vm.box = "bento/centos-7.5"
  kube3.vm.define "kube3"
  kube3.vm.network "public_network", use_dhcp_assigned_default_route: true
    kube3.vm.provider "virtualbox" do |v|
		v.memory = 2048
		v.cpus = 4
	end
end
```

開始安裝
```
Vagrant up --provider=virtualbox
```

同樣方式安裝 kube2 兩台 virtualbox 
```
E:\vagrant\kube2>Vagrant up --provider=virtualbox
Bringing machine 'kube2' up with 'virtualbox' provider..
==> kube2: Importing base box 'bento/centos-7.5'...
Progress: 50%
```

裝完後看狀況
```
vagrant global-status
```

```
id       name   provider   state   directory
-----------------------------------------------------------------------
f6ace63  kube1  virtualbox running C:/vagrant/kube1
5b94637  kube2  virtualbox running C:/vagrant/kube2
0da4a16  kube3  virtualbox running C:/vagrant/kube3

```
ssh 到 kube1
```
vagrant ssh f6ace63
``` 
改一下 vagrant 帳號的密碼 ( defailt 密碼是 vagrant)

利用 mRemoteNG ssh 到 127.0.0.1:2222 , 127.0.0.1:2200 , 127.0.0.1:2201   這樣比較好操作

進入 kube1  安裝用 Ansible 來部屬 kubemaster 
```
sudo yum -y install epel-release
sudo yum -y update
sudo yum -y install ansible python-netaddr git
sudo cat >>/etc/hosts<<EOF
192.168.0.164 kube1 master1
192.168.0.165 kube2 node1
192.168.0.166 kube3 node2
EOF
```

```
sudo cat >>/etc/ansible/hosts<<EOF
[kube1]
192.168.0.164
[kube2]
192.168.0.165
[kube3]
192.168.0.166
EOF
```

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
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.155
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.156
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.157
```
for test connect

ansible all -m ping

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
