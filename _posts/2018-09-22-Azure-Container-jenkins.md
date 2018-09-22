---
layout: post
title: Azure Container exec jenkins
date: 2018-09-22
tags: azure docker
---

## Azure Container exec jenkins


https://www.katacoda.com/courses/cloud/deploying-container-instances

Username: user-tplu@azurelabs.katacoda.com

Password: dUOAiAmdOEXg0840


```
az login -u user-tplu@azurelabs.katacoda.com -p dUOAiAmdOEXg0840

az container create -g user-tplu --name jenkins  --image docker.io/jenkins/jenkins --ip-address public  --ports 8080 5000

az container list -o table

az container exec -g user-tplu --name jenkins --exec-command "/bin/sh "

cat /var/jenkins_home/secrets/initialAdminPassword
```

delete
```
az container delete -g user-tplu --name jenkins
```
