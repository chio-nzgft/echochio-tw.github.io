---
layout: post
title: docker machine 
date: 2018-08-30
tags: docker
---

## docker-machine

install Docker Machine 

If you are running on Linux:

```
$ base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

If you are running with Windows with Git BASH:
```
$ base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  mkdir -p "$HOME/bin" &&
  curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" &&
  chmod +x "$HOME/bin/docker-machine.exe"
```

build virtualBox Docker engine
```
docker-machine create -d virtualbox test
```

env 載入環境變數進行 docker 操作
```
eval $(docker-machine env test)
```

```
docker-machine ls

NAME       ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
test       *        virtualbox   Running   tcp://192.168.99.101:2376           v1.10.3
(
ACTIVE:  * 表示正在操作的 Docker engine
STATE: Docker engine status 
URL: engine ip
SWARM: 是否為 SWARM master or node，
DOCKER: 建立 Docker 時所安裝的 Docker 版本
)

```

```
docker-machine create \
--driver generic \
--generic-ip-address=11.22.33.44 \
--generic-ssh-key=~/.ssh/id_rsa \
ubuntuvm

eval "$(docker-machine env ubuntuvm)"
```

DOCKER 版本 升級
```
docker-machine upgrade
```


```
# docker-machine rm test
About to remove test
Are you sure? (y/n): Y
Successfully removed test
```

azure 
```
docker-machine create --driver azure --azure-subscription-id YOUR_SUBSCR --azure-location eastus --azure-open-port 80 --azure-size "Standard_DS2_v2" azure-docker-vm1

(YOUR_SUBSCR 帳戶識別碼
 eastus 地區機房，eastus代表美國東部
 --azure-open-port 80 宣告防火牆開通80連接埠)

docker-machine ssh azure-docker-vm1
sudo usermod -aG docker $USER
(將 目前使用者加入到 docker 群組 就不必 sudo)
```

```
# docker-machine env azure-docker-vm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://40.68.253.11:2376"
export DOCKER_CERT_PATH="/Users/user/.docker/machine/machines/machine"
export DOCKER_MACHINE_NAME="machine"
# Run this command to configure your shell:
# eval $(docker-machine env azure-docker-vm1)

# docker-machine env
# docker run -d -p 80:80 --restart=always nginx
# docker-machine ip azure-docker-vm1
(就會看到 外部IP ....用瀏覽器就看到 nginx)

```
