---
layout: post
title: Installing Weave Scope on Kubernetes
date: 2018-09-21
tags: docker
---

```
kubectl create -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml'

kubectl get pods -n weave

pod=$(kubectl get pod -n weave --selector=name=weave-scope-app -o jsonpath={.items..metadata.name})

kubectl expose pod $pod -n weave --external-ip="172.17.0.71" --port=4040 --target-port=4040

kubectl get svc --all-namespaces

```

```
NAMESPACE     NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default       kubernetes                         ClusterIP   10.96.0.1       <none>        443/TCP         9m
kube-system   kube-dns                           ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   9m
weave         weave-scope-app                    ClusterIP   10.111.254.86   <none>        80/TCP          8m
weave         weave-scope-app-545ffb956d-7x95x   ClusterIP   10.96.190.131   172.17.0.71   4040/TCP        7m
```

<img src="/images/posts/docker/1.png">
