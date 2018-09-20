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
Vagrant-centos 在 (用此 script 可以直接跳到 .....ansible-playbook site.yaml)

```
https://github.com/echochio-tw/kubeadm-ansible/tree/master/Vagrantfile-centos
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
  config.vm.define :master, primary: true do |master|
    master.vm.network :forwarded_port, host: 8001, guest: 8001
	master.vm.network :forwarded_port, host: 32162, guest: 32162
	master.vm.network :forwarded_port, host: 30716, guest: 30716
	master.vm.network :forwarded_port, host: 2201, guest: 22, id: "ssh", auto_correct: true
    master.vm.network "private_network", ip: "192.168.22.164"
	master.vm.hostname = "master"
	master.vm.provision "shell", path: "bootstrap.sh"
	master.vm.provision "shell", path: "install_ansible.sh"
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
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
swapoff -a
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

```
跳過 host_key_checking
sed -i 's/#host_key_checking = False/host_key_checking = False/g' /etc/ansible/ansible.cfg
```

```
產生 ssh key
rm -rf /root/.ssh
ssh-keygen -t rsa -f /root/.ssh/id_rsa -q -N ""
```

```
複製 key .....
yum install -y sshpass
sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub -o StrictHostKeyChecking=no root@master
sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub -o StrictHostKeyChecking=no root@node1
sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub -o StrictHostKeyChecking=no root@node2
```
```
抓 kubeadm-ansible.git
git clone https://github.com/echochio-tw/kubeadm-ansible.git
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
network: calico

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

```
[root@master kubeadm-ansible]# kubectl proxy &
Starting to serve on 127.0.0.1:8001
```

```
[root@master kubeadm-ansible]# kubectl -n kube-system edit service kubernetes-dashboard
```
加入 externalIPs (eth0 那個)
``` 
spec:
  clusterIP: 10.111.153.203
  externalIPs:
  - 10.0.2.15
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
```

```
[root@master ~]#  kubectl -n kube-system get service kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.101.53.84   10.0.2.15     443/TCP         1h
```

用 host firefox 開 ....
```
https://127.0.0.1/#!/login
```

設定 token
```
cat <<EOF | kubectl create -f -
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
EOF

[root@master kubeadm-ansible]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') |grep "admin-token"
Name:         admin-token-64cq9

[root@master kubeadm-ansible]# kubectl -n kube-system describe secret admin-token-64cq9
Name:         admin-token-64cq9
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin
              kubernetes.io/service-account.uid=40c7f40f-bbe4-11e8-b802-0800278bc93f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi02NGNxOSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQwYzdmNDBmLWJiZTQtMTFlOC1iODAyLTA4MDAyNzhiYzkzZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.UilRq1LJoGI5auUHK3IY83dHwIMDKKyLs8SLmzxmmRpki7TSoEuk4w8DHpZPYwQvrK4bSivXv_GOkCz-cFucBe_c4-DtKSiGO_HBKhLQN3o8YMHge73rwuPG2tV8wO_3OBSLDkZPfDPVj7MFjlK2d_V076m0xxgnS8P8eXtkTY13pYDhP_CA5oQMliAZ4eM68HLfz8EzLFspJOIM-_Fu23IVBUU2Ut3HKjphYuu6SOKR8F4yNfhbPC3F7U6uxgMOzXxOIimhbUTqqbAroSkQee3cqQP7Xa3DSWE4ILd5rhFS9v7M6_FP7RYoGvgd7M06hvuMNcz4V8TB4MD02DbMdA
[root@master kubeadm-ansible]#
```

<img src="/images/posts/kubernetes/p1.png">

<img src="/images/posts/kubernetes/p2.png">

<img src="/images/posts/kubernetes/p3.png">

<img src="/images/posts/kubernetes/p4.png">

<img src="/images/posts/kubernetes/p5.png">

<img src="/images/posts/kubernetes/p6.png">

<img src="/images/posts/kubernetes/p7.png">

<img src="/images/posts/kubernetes/p8.png">

<img src="/images/posts/kubernetes/p9.png">

yml  file Deploy
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
``` 

yml  file Service
```
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  ports:
    - name: http
      port: 80
      nodePort: 30716
  selector:
    app: nginx
  type: NodePort
```
yml file job (斷線自動起 ... 可不設定)
```
apiVersion: batch/v1
kind: Job
metadata:
  name: nginx
spec:
  template:
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        command:
          - sleep
          - "30"
      restartPolicy: Never
```


shell make Deploy
```
cat << 'EOF' >> nginx.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

[root@master ~]#  kubectl create -f nginx.yaml
deployment "nginx" created
[root@master ~]# kubectl get rs,pod,deployment
[root@master ~]# kubectl get rs,pod,deployment
NAME                                             DESIRED   CURRENT   READY     AGE
replicaset.extensions/nginx-966857787            2         2         2         8m

NAME                                 READY     STATUS    RESTARTS   AGE
pod/nginx-966857787-jbpj7            1/1       Running   1          8m
pod/nginx-966857787-vpzpd            1/1       Running   1          8m

NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx            2         2         2            2           8m
````

shell make Service 
```
[root@master ~]#  kubectl expose deployment nginx --type=NodePort
service/nginx exposed
[root@master ~]# kubectl describe services nginx
Name:                     nginx
Namespace:                default
Labels:                   run=nginx
Annotations:              <none>
Selector:                 run=nginx
Type:                     NodePort
IP:                       10.101.98.236
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30716/TCP
Endpoints:                10.244.104.3:80,10.244.166.130:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@master ~]# kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx	         NodePort    10.101.98.236   <none>        80:30716/TCP     16s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          21h
```
<img src="/images/posts/kubernetes/p10.png">
