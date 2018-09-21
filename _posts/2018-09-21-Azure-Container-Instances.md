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
C:\Users\user>az container create -g user-rzhr --name nginx  --image nginx:1.11 --ip-address public
```

```
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "nginx:1.11",
      "instanceView": {
        "currentState": {
          "detailStatus": "",
          "exitCode": null,
          "finishTime": null,
          "startTime": "2018-09-21T05:53:54+00:00",
          "state": "Running"
        },
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:39+00:00",
            "lastTimestamp": "2018-09-21T05:53:39+00:00",
            "message": "pulling image \"nginx:1.11\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:51+00:00",
            "lastTimestamp": "2018-09-21T05:53:51+00:00",
            "message": "Successfully pulled image \"nginx:1.11\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:53+00:00",
            "lastTimestamp": "2018-09-21T05:53:53+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:54+00:00",
            "lastTimestamp": "2018-09-21T05:53:54+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "livenessProbe": null,
      "name": "nginx",
      "ports": [
        {
          "port": 80,
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
  "id": "/subscriptions/8640e4e6-7133-434a-883e-027266896b9d/resourceGroups/user-rzhr/providers/Microsoft.ContainerInstance/containerGroups/nginx",
  "imageRegistryCredentials": null,
  "instanceView": {
    "events": [],
    "state": "Running"
  },
  "ipAddress": {
    "dnsNameLabel": null,
    "fqdn": null,
    "ip": "40.121.47.70",
    "ports": [
      {
        "port": 80,
        "protocol": "TCP"
      }
    ]
  },
  "location": "eastus",
  "name": "nginx",
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
        "image": "nginx:1.11",
        "instanceView": null,
        "livenessProbe": null,
        "name": "nginx",
        "ports": [
          {
            "port": 80,
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
    "id": "/subscriptions/8640e4e6-7133-434a-883e-027266896b9d/resourceGroups/user-rzhr/providers/Microsoft.ContainerInstance/containerGroups/nginx",
    "imageRegistryCredentials": null,
    "instanceView": null,
    "ipAddress": {
      "dnsNameLabel": null,
      "fqdn": null,
      "ip": "40.121.47.70",
      "ports": [
        {
          "port": 80,
          "protocol": "TCP"
        }
      ]
    },
    "location": "eastus",
    "name": "nginx",
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
```
C:\Users\user>az container logs -g user-rzhr --name nginx
```
```
C:\Users\user>az container delete --resource-group user-rzhr --name nginx
Are you sure you want to perform this operation? (y/n): y
```

```
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "nginx:1.11",
      "instanceView": {
        "currentState": {
          "detailStatus": "",
          "exitCode": null,
          "finishTime": null,
          "startTime": "2018-09-21T05:53:54+00:00",
          "state": "Running"
        },
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:39+00:00",
            "lastTimestamp": "2018-09-21T05:53:39+00:00",
            "message": "pulling image \"nginx:1.11\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:51+00:00",
            "lastTimestamp": "2018-09-21T05:53:51+00:00",
            "message": "Successfully pulled image \"nginx:1.11\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:53+00:00",
            "lastTimestamp": "2018-09-21T05:53:53+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2018-09-21T05:53:54+00:00",
            "lastTimestamp": "2018-09-21T05:53:54+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "livenessProbe": null,
      "name": "nginx",
      "ports": [
        {
          "port": 80,
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
-rzhr/providers/Microsoft.ContainerInstance/containerGroups/nginx",
  "imageRegistryCredentials": null,
  "instanceView": {
    "events": [],
    "state": "Running"
  },
  "ipAddress": {
    "dnsNameLabel": null,
    "fqdn": null,
    "ip": "40.121.47.70",
    "ports": [
      {
        "port": 80,
        "protocol": "TCP"
      }
    ]
  },
  "location": "eastus",
  "name": "nginx",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "user-rzhr",
  "restartPolicy": "Always",
  "tags": {},
  "type": "Microsoft.ContainerInstance/containerGroups",
  "volumes": null
}

```
