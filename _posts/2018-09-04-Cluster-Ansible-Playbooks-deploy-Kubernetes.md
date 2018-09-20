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

```
rm -rf /etc/haproxy/haproxy.cfg
cat << 'EOF' >> /etc/haproxy/haproxy.cfg
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  tune.ssl.default-dh-param 2048

defaults
  log global
  mode http
  option dontlognull
  timeout connect 5000ms
  timeout client 1800000ms
  timeout server 1800000ms

listen stats
    bind :9090
    mode http
    balance
    stats uri /haproxy_stats
    stats auth admin:admin123
    stats admin if TRUE

frontend kube-apiserver-https
   mode tcp
   bind :8443
   default_backend kube-apiserver-backend

backend kube-apiserver-backend
    mode tcp
    server kube-apiserver1 k8s-m1:6443 check
    server kube-apiserver2 k8s-m2:6443 check
EOF
```
k8s-m1
```
rm -rf /etc/keepalived/keepalived.conf
cat << 'EOF' >> /etc/keepalived/keepalived.conf
vrrp_script chk_haproxy {
    script "systemctl is-active haproxy"
}

vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 1
    priority 100
    virtual_ipaddress {
        192.168.0.220
    }
    track_script {
        chk_haproxy
    }
}
EOF
```
k8s-m2
```
rm -rf /etc/keepalived/keepalived.conf
cat << 'EOF' >> /etc/keepalived/keepalived.conf
vrrp_script chk_haproxy {
    script "systemctl is-active haproxy"
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 1
    priority 100
    virtual_ipaddress {
        192.168.0.220
    }
    track_script {
        chk_haproxy
    }
}
EOF
```
```
systemctl enable haproxy
systemctl enable keepalived
systemctl start haproxy
systemctl start keepalived
```
chek HA ip 192.168.0.220
```
[root@master1 ~]# ping 192.168.0.220 -c 5
PING 192.168.0.220 (192.168.0.220) 56(84) bytes of data.
64 bytes from 192.168.0.220: icmp_seq=1 ttl=64 time=0.400 ms
64 bytes from 192.168.0.220: icmp_seq=2 ttl=64 time=0.313 ms
64 bytes from 192.168.0.220: icmp_seq=3 ttl=64 time=0.590 ms
64 bytes from 192.168.0.220: icmp_seq=4 ttl=64 time=0.325 ms
64 bytes from 192.168.0.220: icmp_seq=5 ttl=64 time=0.664 ms

--- 192.168.0.220 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.313/0.458/0.664/0.144 ms
```

# k8s-m1 ...
```
sudo yum -y install epel-release
sudo yum -y update
sudo yum -y install ansible python-netaddr git cowsay
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
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
```
[root@k8s-m2 ~]#  kubectl get po -n kube-system
NAME                                   READY     STATUS    RESTARTS   AGE
calico-node-6tbqd                      2/2       Running   0          10h
calico-node-b4fml                      2/2       Running   0          10h
calico-node-km5km                      2/2       Running   0          10h
calico-node-txpqw                      2/2       Running   0          10h
coredns-6d98b868c7-zlqjp               1/1       Running   0          10h
elasticsearch-logging-0                1/1       Running   1          5m
fluentd-es-5lqz4                       1/1       Running   0          5m
fluentd-es-mgmqt                       1/1       Running   0          5m
fluentd-es-pbggw                       1/1       Running   0          5m
fluentd-es-r6lh7                       1/1       Running   0          5m
kibana-logging-56c4d58dcd-jgrjv        1/1       Running   0          5m
kube-apiserver-k8s-m1                  1/1       Running   0          10h
kube-apiserver-k8s-m2                  1/1       Running   0          10h
kube-controller-manager-k8s-m1         1/1       Running   0          10h
kube-controller-manager-k8s-m2         1/1       Running   0          10h
kube-haproxy-k8s-m1                    1/1       Running   0          10h
kube-haproxy-k8s-m2                    1/1       Running   0          10h
kube-keepalived-k8s-m1                 1/1       Running   0          10h
kube-keepalived-k8s-m2                 1/1       Running   0          10h
kube-proxy-d5wpg                       1/1       Running   0          10h
kube-proxy-gl5r6                       1/1       Running   0          10h
kube-proxy-lhl5w                       1/1       Running   0          10h
kube-proxy-mk5bl                       1/1       Running   0          10h
kube-scheduler-k8s-m1                  1/1       Running   0          10h
kube-scheduler-k8s-m2                  1/1       Running   0          10h
kubernetes-dashboard-6948bdb78-m2nv5   1/1       Running   0          6m

```
# dashboard 
```
kubectl get po,svc -n kube-system -l k8s-app=kubernetes-dashboard
```
```
NAME                                       READY     STATUS    RESTARTS   AGE
pod/kubernetes-dashboard-6948bdb78-mc9q2   1/1       Running   0          52m

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes-dashboard   ClusterIP   10.111.153.203   <none>        443/TCP   52m
```
```
kubectl -n kube-system edit service kubernetes-dashboard 
```
加入 externalIPs
```
spec:
  clusterIP: 10.111.153.203
  externalIPs:
  - 192.168.0.155
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443

```
```
kubectl get po,svc -n kube-system -l k8s-app=kubernetes-dashboard
```
```
NAME                                       READY     STATUS    RESTARTS   AGE
pod/kubernetes-dashboard-6948bdb78-mc9q2   1/1       Running   0          1h

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP     PORT(S)   AGE
service/kubernetes-dashboard   ClusterIP   10.111.153.203   192.168.0.155   443/TCP   1h
```

用 firefox 開 https://192.168.0.155/

login with token

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

[root@master1 kube-ansible]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
Name:         admin-token-l4976
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin
              kubernetes.io/service-account.uid=0283efe0-bba9-11e8-b412-0800278bc93f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1428 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1sNDk3NiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjAyODNlZmUwLWJiYTktMTFlOC1iNDEyLTA4MDAyNzhiYzkzZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.CSaiovNKe_IJPPD1FcZ9XuNuAOz5Ugezg05TeGTwQyMT4nMGDqZVAF4028y3gYIPouQ8Ml9ixOR5B5dTLH7Mvhrk4qfD4HU7PkPHKih78xJBbmn8-1CQLDmnoq54a_WSfn7ciotiukspa7Lue2gxIZmTG8RR8W-qvOW5kRw4weSE69wVHRonAG8GtA4uMbqC0vdtjAElUaMXWZkS52quiOM9rRswU2Sd909PsGducrFuu5doZUqsMu-g-swPkj1-F_tojeztEm_BvINYiojg3RJXcg_u52-KyhRXH4D-qeAgX9bLsQVZQxQo5K8PAGPwCWeFeaj2t6osUBdZKcfKAQ

```

check ha
```
export PKI="/etc/kubernetes/pki/etcd"
ETCDCTL_API=3 etcdctl \
    --cacert=${PKI}/etcd-ca.pem \
    --cert=${PKI}/etcd.pem \
    --key=${PKI}/etcd-key.pem \
    --endpoints="https://127.0.0.1:2379" \
    member list
```

```
43d16080ce52a959, started, master1, https://192.168.0.155:2380, https://192.168.0.155:2379
a4e53d585466d56d, started, master2, https://192.168.0.156:2380, https://192.168.0.156:2379
```
start nginx
```
kubectl run nginx --image nginx --restart=Never --port 80
kubectl expose pod nginx --port 80 --type NodePort
kubectl get po,svc
```
check nginx url
```
curl xxx.xxx.xxx:31780
```
```
kubectl get secret,sa,role,rolebinding,services,deployments,pod --all-namespaces
```
```
kubectl get secret,sa,role,rolebinding,services,deployments  --all-namespaces
```

rollback install
```
ansible-playbook -i inventory/hosts.ini reset-cluster.yml
```
