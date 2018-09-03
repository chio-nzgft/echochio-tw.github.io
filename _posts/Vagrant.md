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
改一下 vagrant 帳號的密碼

利用 mRemoteNG ssh 到 127.0.0.1:2222 , 127.0.0.1:2200 , 127.0.0.1:2201   這樣比較好操作

進入 kube1  安裝用 Ansible 來部屬 kubemaster 
```
sudo yum -y install epel-release
sudo yum -y update
sudo yum -y install ansible python-netaddr git
cat >>/etc/hosts<<EOF
192.168.0.155 kube1
192.168.0.156 kube2
192.168.0.157 kube3
EOF
```

```
cat >>/etc/ansible/hosts<<EOF
[kube1]
192.168.0.155
[kube2]
192.168.0.156
[kube3]
192.168.0.157
EOF
```

/etc/ansible/ansible.cfg
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
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.101
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.102
```

for test connect
```
192.168.1.101 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.1.102 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

```
yum install -y unzip
wget https://github.com/containerum/letskube/archive/master.zip
unzip master.zip
cd letskube-master
```

inventory
```
[all]
m1 ansible_user=root ansible_host=192.168.1.101 ansible_port=22 ip_internal=10.0.0.1
s1 ansible_user=root ansible_host=192.168.1.102 ansible_port=22 ip_internal=10.0.0.2
[masters]
m1
[slaves]
s1
[kubectl]
m1
```

```
 ansible-playbook bootstrap.yaml -i inventory -v
 
```

```
kubectl get nodes
```
