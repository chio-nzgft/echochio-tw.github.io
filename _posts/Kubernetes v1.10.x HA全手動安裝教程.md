host info
```
192.168.0.175 k8s-m1 master1
192.168.0.176 k8s-m2 master2
192.168.0.177 k8s-n1 node1
192.168.0.178 k8s-n2 node2
192.168.0.222 cluster IP
````

all node
```
systemctl stop firewalld && systemctl disable firewalld
setenforce 0
sed -i --follow-symlinks 's/SELINUX=permissive/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sudo cat >>/etc/hosts<<EOF
192.168.0.175 k8s-m1
192.168.0.176 k8s-m2
192.168.0.177 k8s-n1
192.168.0.178 k8s-n2
EOF
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
curl -fsSL "https://get.docker.com/" | sh
systemctl enable docker && systemctl start docker
cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p /etc/sysctl.d/k8s.conf
swapoff -a && sysctl -w vm.swappiness=0
export KUBE_URL="https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64"
wget "${KUBE_URL}/kubelet" -O /usr/local/bin/kubelet
chmod +x /usr/local/bin/kubelet
```
# node 請忽略下載 kubectl
```
wget "${KUBE_URL}/kubectl" -O /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubectl
mkdir -p /opt/cni/bin && cd /opt/cni/bin
export CNI_URL="https://github.com/containernetworking/plugins/releases/download"
wget -qO- "${CNI_URL}/v0.6.0/cni-plugins-amd64-v0.6.0.tgz" | tar -zx
```
k8s-m1需要安裝CFSSL工具，這將會用來建立 TLS Certificates。
```
export CFSSL_URL="https://pkg.cfssl.org/R1.2"
wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl
wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```
build Etcd CA keys & Certificates in k8s-m1
```
mkdir -p /etc/etcd/ssl && cd /etc/etcd/ssl
export PKI_URL="https://kairen.github.io/files/manual-v1.10/pki"
wget "${PKI_URL}/ca-config.json" "${PKI_URL}/etcd-ca-csr.json"
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca
wget "${PKI_URL}/etcd-csr.json"
cfssl gencert \
  -ca=etcd-ca.pem \
  -ca-key=etcd-ca-key.pem \
  -config=ca-config.json \
  -hostname=127.0.0.1,192.168,192.0.175,192.168,192.0.176 \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare etcd
rm -rf *.json *.csr
```

ls /etc/etcd/ssl

```
etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem
```

```
for NODE in k8s-m2; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p /etc/etcd/ssl"
    for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
      scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
    done
  done
```

k8s-m1 build Kubernetes pki (192.168.0.222 cluster IP)
```
mkdir -p /etc/kubernetes/pki && cd /etc/kubernetes/pki
export PKI_URL="https://kairen.github.io/files/manual-v1.10/pki"
export KUBE_APISERVER="https://192.168.0.222:6443"
wget "${PKI_URL}/ca-config.json" "${PKI_URL}/ca-csr.json"
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

 ls ca*.pem
```
ca-key.pem  ca.pem
```

 Kubernetes cluster IP 192.168.0.222
 
```
wget "${PKI_URL}/apiserver-csr.json"
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.96.0.1,192.168.0.222,127.0.0.1,kubernetes.default \
  -profile=kubernetes \
  apiserver-csr.json | cfssljson -bare apiserver
```
ls apiserver*.pem
```
apiserver-key.pem  apiserver.pem 
```
Front Proxy CA:
```
wget "${PKI_URL}/front-proxy-ca-csr.json"
cfssl gencert \
  -initca front-proxy-ca-csr.json | cfssljson -bare front-proxy-ca
```

 ls front-proxy-ca*.pem

 ```
front-proxy-ca-key.pem  front-proxy-ca.pem
```

front-proxy-client CA:

```
wget "${PKI_URL}/front-proxy-client-csr.json"
cfssl gencert \
  -ca=front-proxy-ca.pem \
  -ca-key=front-proxy-ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  front-proxy-client-csr.json | cfssljson -bare front-proxy-client
```


ls front-proxy-client*.pem
```
front-proxy-client-key.pem  front-proxy-client.pem
```

Admin Certificate
```
wget "${PKI_URL}/admin-csr.json"
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```
ls admin*.pem
```
admin-key.pem  admin.pem
```

admin set cluster
```
kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=../admin.conf
```

admin set credentials
```
kubectl config set-credentials kubernetes-admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=../admin.conf
```

admin set context
```
kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes \
    --user=kubernetes-admin \
    --kubeconfig=../admin.conf
```

admin set default context
```
kubectl config use-context kubernetes-admin@kubernetes \
    --kubeconfig=../admin.conf
```
Controller Manager Certificate

IP 不同，需修改manager-csr.json的hosts
```
wget "${PKI_URL}/manager-csr.json"
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  manager-csr.json | cfssljson -bare controller-manager
```

 ls controller-manager*.pem
```
controller-manager-key.pem  controller-manager.pem
```
controller-manager set cluster
```
kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=../controller-manager.conf
```
controller-manager set credentials
```
kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=controller-manager.pem \
    --client-key=controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=../controller-manager.conf
```

controller-manager set context
```
kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=../controller-manager.conf
```
controller-manager set default context
```
kubectl config use-context system:kube-controller-manager@kubernetes \
    --kubeconfig=../controller-manager.conf
```

Scheduler Certificate 

 IP 不同，需要修改scheduler-csr.json的hosts
```
wget "${PKI_URL}/scheduler-csr.json"
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  scheduler-csr.json | cfssljson -bare scheduler
```
ls scheduler*.pem
```
scheduler-key.pem  scheduler.pem
```
scheduler set cluster
```
kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=../scheduler.conf
```

scheduler set credentials
```
kubectl config set-credentials system:kube-scheduler \
    --client-certificate=scheduler.pem \
    --client-key=scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=../scheduler.conf
```
scheduler set context
```
kubectl config set-context system:kube-scheduler@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=../scheduler.conf
```
scheduler use default context
```
kubectl config use-context system:kube-scheduler@kubernetes \
    --kubeconfig=../scheduler.conf
```
Master Kubelet Certificate
```
wget "${PKI_URL}/kubelet-csr.json"
for NODE in k8s-m1 k8s-m2; do
    echo "--- $NODE ---"
    cp kubelet-csr.json kubelet-$NODE-csr.json;
    sed -i "s/\$NODE/$NODE/g" kubelet-$NODE-csr.json;
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=$NODE \
      -profile=kubernetes \
      kubelet-$NODE-csr.json | cfssljson -bare kubelet-$NODE
  done
```
ls kubelet*.pem
```
kubelet-k8s-m1-key.pem  kubelet-k8s-m1.pem  kubelet-k8s-m2-key.pem  kubelet-k8s-m2.pem
```
據節點修改-HostName與$NODE

複製kubelet憑證至其他master：
```
for NODE in k8s-m2; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p /etc/kubernetes/pki"
    for FILE in kubelet-$NODE-key.pem kubelet-$NODE.pem ca.pem; do
      scp /etc/kubernetes/pki/${FILE} ${NODE}:/etc/kubernetes/pki/${FILE}
    done
 done
```


```
for NODE in k8s-m1 k8s-m2; do
    echo "--- $NODE ---"
    ssh ${NODE} "cd /etc/kubernetes/pki && \
      kubectl config set-cluster kubernetes \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=${KUBE_APISERVER} \
        --kubeconfig=../kubelet.conf && \
      kubectl config set-cluster kubernetes \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=${KUBE_APISERVER} \
        --kubeconfig=../kubelet.conf && \
      kubectl config set-credentials system:node:${NODE} \
        --client-certificate=kubelet-${NODE}.pem \
        --client-key=kubelet-${NODE}-key.pem \
        --embed-certs=true \
        --kubeconfig=../kubelet.conf && \
      kubectl config set-context system:node:${NODE}@kubernetes \
        --cluster=kubernetes \
        --user=system:node:${NODE} \
        --kubeconfig=../kubelet.conf && \
      kubectl config use-context system:node:${NODE}@kubernetes \
        --kubeconfig=../kubelet.conf && \
      rm kubelet-${NODE}.pem kubelet-${NODE}-key.pem"
  done
```

Service Account Key
```
openssl genrsa -out sa.key 2048
openssl rsa -in sa.key -pubout -out sa.pub
```
ls sa.*
```
sa.key  sa.pub
```

```
rm -rf *.json *.csr scheduler*.pem controller-manager*.pem admin*.pem kubelet*.pem
for NODE in k8s-m2; do
    echo "--- $NODE ---"
    for FILE in $(ls /etc/kubernetes/pki/); do
      scp /etc/kubernetes/pki/${FILE} ${NODE}:/etc/kubernetes/pki/${FILE}
    done
  done
```

```
for NODE in k8s-m2; do
    echo "--- $NODE ---"
    for FILE in admin.conf controller-manager.conf scheduler.conf; do
      scp /etc/kubernetes/${FILE} ${NODE}:/etc/kubernetes/${FILE}
    done
  done
```

Kubernetes Masters

all master:
```
export CORE_URL="https://kairen.github.io/files/manual-v1.10/master"
mkdir -p /etc/kubernetes/manifests && cd /etc/kubernetes/manifests
for FILE in kube-apiserver kube-controller-manager kube-scheduler haproxy keepalived etcd etcd.config; do
    wget "${CORE_URL}/${FILE}.yml.conf" -O ${FILE}.yml
    if [ ${FILE} == "etcd.config" ]; then
      mv etcd.config.yml /etc/etcd/etcd.config.yml
      sed -i "s/\${HOSTNAME}/${HOSTNAME}/g" /etc/etcd/etcd.config.yml
      sed -i "s/\${PUBLIC_IP}/$(hostname -i)/g" /etc/etcd/etcd.config.yml
    fi
done
```

ls /etc/kubernetes/manifests 

# (need change yml IP....)
```
etcd.yml  haproxy.yml  keepalived.yml  kube-apiserver.yml  kube-controller-manager.yml  kube-scheduler.yml
```
 Etcd Key
```
head -c 32 /dev/urandom | base64
```

```
SUpbL4juUYyvxj3/gonV5xVEx8j769/99TSAf8YT/sQ=
```

all master
```
 cat <<EOF > /etc/kubernetes/encryption.yml
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: SUpbL4juUYyvxj3/gonV5xVEx8j769/99TSAf8YT/sQ=
      - identity: {}
EOF
cat <<EOF > /etc/kubernetes/audit-policy.yml
apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:- level: Metadata
EOF
```

```
mkdir -p /etc/haproxy/
wget "${CORE_URL}/haproxy.cfg" -O /etc/haproxy/haproxy.cfg
mkdir -p /etc/systemd/system/kubelet.service.d
wget "${CORE_URL}/kubelet.service" -O /lib/systemd/system/kubelet.service
wget "${CORE_URL}/10-kubelet.conf" -O /etc/systemd/system/kubelet.service.d/10-kubelet.conf
mkdir -p /var/lib/kubelet /var/log/kubernetes /var/lib/etcd
systemctl enable kubelet.service && systemctl start kubelet.service
```

```
watch netstat -ntlp

Active Internet connections (only servers)Proto Recv-Q Send-Q 
```

```
cp /etc/kubernetes/admin.conf ~/.kube/config
kubectl get cs
kubectl get node
kubectl -n kube-system get po
```

```
kubectl apply -f "${CORE_URL}/apiserver-to-kubelet-rbac.yml.conf"
```

```
kubectl -n kube-system logs -f kube-scheduler-k8s-m2Error from server (Forbidden): Forbidden (user=kube-apiserver, verb=get, resource=nodes, subresource=proxy) ( pods/log kube-scheduler-k8s-m2)
```

```
kubectl -n kube-system logs -f kube-scheduler-k8s-m2
```

```
kubectl taint nodes node-role.kubernetes.io/master="":NoSchedule --all
```
k8s-m1 build BOOTSTRAP_TOKEN
```
cd /etc/kubernetes/pki
export TOKEN_ID=$(openssl rand 3 -hex)
export TOKEN_SECRET=$(openssl rand 8 -hex)
export BOOTSTRAP_TOKEN=${TOKEN_ID}.${TOKEN_SECRET}
export KUBE_APISERVER="https://192.168.0.222:6443"
```

bootstrap set cluster
```
kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=${KUBE_APISERVER} \
    --kubeconfig=../bootstrap-kubelet.conf
```

bootstrap set credentials
```
kubectl config set-credentials tls-bootstrap-token-user \
    --token=${BOOTSTRAP_TOKEN} \
    --kubeconfig=../bootstrap-kubelet.conf
```

bootstrap set context
```
kubectl config set-context tls-bootstrap-token-user@kubernetes \
    --cluster=kubernetes \
    --user=tls-bootstrap-token-user \
    --kubeconfig=../bootstrap-kubelet.conf
```
bootstrap use default context
```
kubectl config use-context tls-bootstrap-token-user@kubernetes \
    --kubeconfig=../bootstrap-kubelet.conf
```

Kubernetes Nodes 

k8-m1 scp to all node
```
cd /etc/kubernetes/pki
for NODE in k8s-n1 k8s-n2; do
    echo "--- $NODE ---"
    ssh ${NODE} "mkdir -p /etc/kubernetes/pki/"
    ssh ${NODE} "mkdir -p /etc/etcd/ssl"
	   for FILE in etcd-ca.pem etcd.pem etcd-key.pem; do
		scp /etc/etcd/ssl/${FILE} ${NODE}:/etc/etcd/ssl/${FILE}
	   done
       for FILE in pki/ca.pem pki/ca-key.pem bootstrap-kubelet.conf; do
        scp /etc/kubernetes/${FILE} ${NODE}:/etc/kubernetes/${FILE}
       done
done
```

node 下載kubelet.service相關文件來管理 kubelet：
```
export CORE_URL="https://kairen.github.io/files/manual-v1.10/node"
mkdir -p /etc/systemd/system/kubelet.service.d
wget "${CORE_URL}/kubelet.service" -O /lib/systemd/system/kubelet.service
wget "${CORE_URL}/10-kubelet.conf" -O /etc/systemd/system/kubelet.service.d/10-kubelet.conf
mkdir -p /var/lib/kubelet /var/log/kubernetes
systemctl enable kubelet.service && systemctl start kubelet.service
```
in master
```
 kubectl get csr
 kubectl get nodes
```

Kubernetes Proxy

k8s-m1 downlaod kube-proxy.yml build Kubernetes Proxy Addon：

```
kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-proxy.yml.conf"
kubectl -n kube-system get po -o wide -l k8s-app=kube-proxy
```

Kubernetes DNS

k8s-m1 download kube-proxy.yml build Kubernetes Proxy Addon： (need Kubernetes Pod Network )
```
kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-dns.yml.conf"
kubectl -n kube-system get po -l k8s-app=kube-dns
```

k8s-m1 download calico.yaml build Calico Network： 
(IP need change calico.yml)
```
kubectl apply -f "https://kairen.github.io/files/manual-v1.10/network/calico.yml.conf"
kubectl -n kube-system get po -l k8s-app=calico-node -o wide
```

k8s-m1 download Calico CLI view Calico nodes:
```
wget https://github.com/projectcalico/calicoctl/releases/download/v3.1.0/calicoctl -O /usr/local/bin/calicoctl
chmod u+x /usr/local/bin/calicoctl
cat <<EOF > ~/calico-rcexport 
ETCD_ENDPOINTS="https://192.168.0.175:2379,https://192.168.0.176:2379export 
ETCD_CA_CERT_FILE="/etc/etcd/ssl/etcd-ca.pem"export 
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"export 
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
EOF
. ~/calico-rc
```
check dns will Running ...
```
kubectl -n kube-system get po -l k8s-app=kube-dns
```

Kubernetes Extra Addons

Dashboard

k8s-m1 use kubectl build kubernetes dashboard ：(need check kubernetes-dashboard.yaml IP)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl -n kube-system get po,svc -l k8s-app=kubernetes-dashboard
```
open-api Cluster Role Binding (for test only)
```
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: open-api
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: system:anonymous
EOF
```
build service account binding cluster-admin role：
```
kubectl -n kube-system create sa dashboard
kubectl create clusterrolebinding dashboard --clusterrole cluster-admin --serviceaccount=kube-system:dashboard
SECRET=$(kubectl -n kube-system get sa dashboard -o yaml | awk '/dashboard-token/ {print $3}')
kubectl -n kube-system describe secrets ${SECRET} | awk '/token:/{print $2}'
```

token，貼到 Kubernetes dashboard。

Heapster

k8s-m1 use kubectl build kubernetes monitor ( need check yml IP)
```
kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/kube-monitor.yml.conf"
kubectl -n kube-system get po,svc
```

Grafana Dashboard
```
https://192.168.0.222:6443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy/
```

Ingress Controller

k8s-m1 use kubectl build Ingress Controller(need check conf file)
```
kubectl create ns ingress-nginx
kubectl apply -f "https://kairen.github.io/files/manual-v1.10/addon/ingress-controller.yml.conf"
kubectl -n ingress-nginx get po
```
test Ingress 
```
kubectl run nginx-dp --image nginx --port 80
kubectl expose deploy nginx-dp --port 80
kubectl get po,svc
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-nginx-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: test.nginx.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-dp
          servicePort: 80
EOF
```
curl test
```
curl 192.168.0.222 -H 'Host: test.nginx.com'
```
fault domain .. will 404
```
curl 192.16.35.10 -H 'Host: test.nginx.com1'
```

k8s-m1 install  Helm tool：
```
wget -qO- https://kubernetes-helm.storage.googleapis.com/helm-v2.8.1-linux-amd64.tar.gz | tar -zx
sudo mv linux-amd64/helm /usr/local/bin/
```
all node install socat
```
apt-get install -y socat
```

init Helm in k8s-m1
```
kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
kubectl -n kube-system get po -l app=helm
helm versionClient: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}Server: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}
```
 
 use helm install jenkins
```
helm install --name demo --set Persistence.Enabled=false stable/jenkins
kubectl get po,svc  -l app=demo-jenkins
```
取 admin 帳號
```
printf $(kubectl get secret --namespace default demo-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```
用瀏覽器瀏覽 .....

刪除 jenkins
```
helm ls
helm delete demo --purge
```
cluster test poweroff k8s-m1 in k8s-m2 check
```
kubectl get cs
```
build nginx for test
```
kubectl run nginx --image nginx --restart=Never --port 80
kubectl get po
```

 
