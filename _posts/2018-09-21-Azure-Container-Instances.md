---
layout: post
title: Azure-Container-Instances
date: 2018-09-21
tags: docker
---

## Azure Container Instances
```
https://www.katacoda.com/courses/cloud/deploying-container-instances
```

```
Username: user-rzhr@azurelabs.katacoda.com
Password: zzfRrho7VjtJzpC0
Resource Group: user-rzhr
Region: "East US"
```

```
C:\Users\user>az login -u user-rzhr@azurelabs.katacoda.com -p zzfRrho7VjtJzpC0
```

```
[
  {
    "cloudName": "AzureCloud",
    "id": "8640e4e6-7133-434a-883e-027266896b9d",
    "isDefault": true,
    "name": "Katacoda Subscription (v1)",
    "state": "Enabled",
    "tenantId": "b3677692-d27b-45e0-8e16-4b739eed0e05",
    "user": {
      "name": "user-rzhr@azurelabs.katacoda.com",
      "type": "user"
    }
  }
]
```

```
C:\Users\user>az container create -g user-rzhr --name openshift-go-web  --image docker.io/echochio/openshift-go-web --ip-address public  --ports 8080
```

```
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "docker.io/echochio/openshift-go-web",
      "instanceView": {
        "currentState": {
          "detailStatus": "",
          "exitCode": null,
          "finishTime": null,
          "startTime": "2018-09-21T06:09:30+00:00",
          "state": "Running"
        },
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T06:09:28+00:00",
            "lastTimestamp": "2018-09-21T06:09:28+00:00",
            "message": "pulling image \"docker.io/echochio/openshift-go-web\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T06:09:30+00:00",
            "lastTimestamp": "2018-09-21T06:09:30+00:00",
            "message": "Successfully pulled image \"docker.io/echochio/openshift
-go-web\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T06:09:30+00:00",
            "lastTimestamp": "2018-09-21T06:09:30+00:00",
            "message": "Created container with id c35bfdf9cf1b451fec0d600efe7902
4c659e4ea9398d1411c457efacfda4fa24",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T06:09:30+00:00",
            "lastTimestamp": "2018-09-21T06:09:30+00:00",
            "message": "Started container with id c35bfdf9cf1b451fec0d600efe7902
4c659e4ea9398d1411c457efacfda4fa24",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "livenessProbe": null,
      "name": "openshift-go-web",
      "ports": [
        {
          "port": 8080,
          "protocol": "TCP"
        }
      ],
      "readinessProbe": null,
      "resources": {
        "limits": null,
        "requests": {
          "cpu": 1.0,
          "memoryInGb": 1.5
        }
      },
      "volumeMounts": null
    }
  ],
  "diagnostics": null,
  "id": "/subscriptions/8640e4e6-7133-434a-883e-027266896b9d/resourceGroups/user
-rzhr/providers/Microsoft.ContainerInstance/containerGroups/openshift-go-web",
  "imageRegistryCredentials": null,
  "instanceView": {
    "events": [],
    "state": "Running"
  },
  "ipAddress": {
    "dnsNameLabel": null,
    "fqdn": null,
    "ip": "40.117.152.212",
    "ports": [
      {
        "port": 8080,
        "protocol": "TCP"
      }
    ]
  },
  "location": "eastus",
  "name": "openshift-go-web",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "user-rzhr",
  "restartPolicy": "Always",
  "tags": {},
  "type": "Microsoft.ContainerInstance/containerGroups",
  "volumes": null
}
```
```
C:\Users\user>az container list
```
```
[
  {
    "containers": [
      {
        "command": null,
        "environmentVariables": [],
        "image": "docker.io/echochio/openshift-go-web",
        "instanceView": null,
        "livenessProbe": null,
        "name": "openshift-go-web",
        "ports": [
          {
            "port": 8080,
            "protocol": "TCP"
          }
        ],
        "readinessProbe": null,
        "resources": {
          "limits": null,
          "requests": {
            "cpu": 1.0,
            "memoryInGb": 1.5
          }
        },
        "volumeMounts": null
      }
    ],
    "diagnostics": null,
    "id": "/subscriptions/8640e4e6-7133-434a-883e-027266896b9d/resourceGroups/us
er-rzhr/providers/Microsoft.ContainerInstance/containerGroups/openshift-go-web",

    "imageRegistryCredentials": null,
    "instanceView": null,
    "ipAddress": {
      "dnsNameLabel": null,
      "fqdn": null,
      "ip": "40.117.152.212",
      "ports": [
        {
          "port": 8080,
          "protocol": "TCP"
        }
      ]
    },
    "location": "eastus",
    "name": "openshift-go-web",
    "osType": "Linux",
    "provisioningState": "Succeeded",
    "resourceGroup": "user-rzhr",
    "restartPolicy": "Always",
    "tags": {},
    "type": "Microsoft.ContainerInstance/containerGroups",
    "volumes": null
  }
]
```

<img src="/images/posts/docker/2.png">

```
C:\Users\user>az container logs -g user-rzhr --name openshift-go-web
```
```
C:\Users\user>az container delete --resource-group user-rzhr --name openshift-go-web
Are you sure you want to perform this operation? (y/n): y
```

