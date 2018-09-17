---
layout: post
title: jenkins docker ansible centos
date: 2018-09-17
tags: docker
---

原始碼 : https://github.com/echochio-tw/jenkins-docker-ansible-centos

```
Vagrantfile 
建立兩個 虛擬機  CD & PROD
CD 開 port 8080(jenkins) , 5000(registry) , 9999 (busybox-go-web-docker)
host: 2201, guest: 22, id: "ssh" , ip: "192.168.50.91"
do shell bootstrap.sh 
do shell ansible-playbook /vagrant/ansible/cd.yml -c local -v

PROD 開 9001 (busybox-go-web-docker)
host: 2202, guest: 22, id: "ssh" , ip: "192.168.50.92"

```

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
  config.vm.define :cd, primary: true do |cd|
    cd.vm.network :forwarded_port, host: 8080, guest: 8080
    cd.vm.network :forwarded_port, host: 5000, guest: 5000
    cd.vm.network :forwarded_port, host: 9999, guest: 9999
    cd.vm.network :forwarded_port, host: 2201, guest: 22, id: "ssh", auto_correct: true
    cd.vm.network "private_network", ip: "192.168.50.91"
    # added dynamic swap file management to prevent "Free Swap Space: 0 B" warning in Jenkins
    cd.vm.provision "shell", path: "bootstrap.sh"
    cd.vm.provision :shell, inline: 'ansible-playbook /vagrant/ansible/cd.yml -c local -v'
    cd.vm.hostname = "cd"
  end
  config.vm.define :prod do |prod|
    prod.vm.network :forwarded_port, host: 2202, guest: 22, id: "ssh", auto_correct: true
    prod.vm.network :forwarded_port, host: 9001, guest: 9001
    prod.vm.network "private_network", ip: "192.168.50.92"
    prod.vm.hostname = "prod"
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

```
bootstrap.sh 
```

安裝 Ansible
```
#!/bin/bash

echo "Installing Ansible..."
yum -y install epel-release
yum -y update
yum -y install ansible python-netaddr git
cp /vagrant/ansible/ansible.cfg /etc/ansible/ansible.cfg
```

```
cd.yml
```
安裝 java , docker , registry , jenkins
```
- hosts: localhost
  remote_user: vagrant
  become: yes
  roles:
    - java
    - docker
    - registry
    - jenkins
```

```
java

- name: Package are present
  yum:
    name: java-1.8.0-openjdk
    state: present
```

```
golang-service.yml
```
給 PROD 虛擬機用 ....

```
- hosts: go-service
  remote_user: vagrant
  become: yes
  roles:
    - docker
    - go-service
```

```
ansible/roles/jenkins/tasks/main.yml
安裝 jenkins (docker)
```

```
ansible/roles/jenkins/defaults/main.yml
set jenkins configs & plugins
```

```
ansible/roles/jenkins/templates/build.xml.j2
books-service 
```

```
ansible/roles/jenkins/templates/deployment.xml.j2
books-service-deployment
```
