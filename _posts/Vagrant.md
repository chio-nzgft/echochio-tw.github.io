win7 install Vagrant

下載 Vagrant 安裝重開
下載 virtualbox 安裝重開
更新 powershell 

```
https://docs.microsoft.com/zh-tw/powershell/wmf/5.1/install-configure
```

設定 
E:\vagrant\kubemaster\Vagrantfile
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.5"
  config.vm.define "kubemaster"
  config.vm.network "private_network", ip: "192.168.1.99"
end
```

開始安裝
```
Vagrant up --provider=virtualbox
```

同樣方式安裝 kube2 & kube3 兩台 virtualbox 
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
id       name       provider   state   directory
---------------------------------------------------------------------------
b625fb2  kubemaster virtualbox running E:/vagrant/kubemaster
b4ab72f  kube2      virtualbox running E:/vagrant/kube2
bfac262  kube3      virtualbox running E:/vagrant/kube3
```
ssh 到 kubemaster
```
vagrant ssh b625fb2
``` 
改一下 vagrant 帳號的密碼

利用 mRemoteNG ssh 到 127.0.0.1:2222 , 127.0.0.1:2201 , 127.0.0.1:2202 這樣比較好操作

進入 kubemaster server  安裝用 Ansible 來部屬 kubemaster 
```
sudo yum -y install epel-release
sudo yum -y update
sudo yum -y install ansible
cat >>/etc/hosts<<EOF
192.168.1.99 kubemaster
192.168.1.109 kube2
192.168.1.167 kube3
EOF
```

```
cat >>/etc/ansible/hosts<<EOF
[kubemaster]
192.168.1.99
[kube2]
192.168.1.109
[kube3]
192.168.1.167
EOF
```

```
ssh-keyscan 192.168.1.99 >> ~/.ssh/known_hosts
ssh-keyscan 192.168.1.109 >> ~/.ssh/known_hosts
ssh-keyscan 192.168.1.167 >> ~/.ssh/known_hosts
```

for test connect
```
export ANSIBLE_HOST_KEY_CHECKING=False
ansible all -m ping --extra-vars "ansible_user=root ansible_password=root"
```

```
192.168.1.167 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.1.109 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.1.99 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

playbook_centos_install_docker.yaml
```
---
- name: Install docker
  gather_facts: No
  hosts: all

  tasks:
    - name: Install yum utils
      yum:
        name: yum-utils
        state: latest

    - name: Install device-mapper-persistent-data
      yum:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      yum:
        name: lvm2
        state: latest

    - name: Add Docker repo
      get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docer-ce.repo
      become: yes

    - name: Enable Docker Edge repo
      ini_file:
        dest: /etc/yum.repos.d/docer-ce.repo
        section: 'docker-ce-edge'
        option: enabled
        value: 0
      become: yes

    - name: Enable Docker Test repo
      ini_file:
        dest: /etc/yum.repos.d/docer-ce.repo
        section: 'docker-ce-test'
        option: enabled
        value: 0
      become: yes

    - name: Install Docker
      package:
        name: docker-ce
        state: latest
      become: yes

    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
      become: yes

    - name: Add user vagrant to docker group
      user:
        name: vagrant
        groups: docker
        append: yes
      become: yes
```

```
ansible-playbook playbook_centos_install_docker.yaml --extra-vars "ansible_user=root ansible_password=root"
```
