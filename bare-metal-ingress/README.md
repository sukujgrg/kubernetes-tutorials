Ingress controller on Bare metal
---

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Steps](#steps)
    + [NGINX Ingress Controller manifests](#nginx-ingress-controller-manifests)
    + [Create namespace, secrets, config maps and rbac](#create-namespace-secrets-config-maps-and-rbac)
    + [Create ingress controller as a Deamonset](#create-ingress-controller-as-a-deamonset)
  * [Deploy the application and Ingress resource](#deploy-the-application-and-ingress-resource)
    + [Deploy the application](#deploy-the-application)
    + [Expose the application with ClusterIP Service](#expose-the-application-with-clusterip-service)
    + [Deploy Ingress resource](#deploy-ingress-resource)
  * [Demo](#demo)

## Introduction
This guide deploys an Ingress Controller and then configure an Ingress resource to route external requestes to kubernetes services.
- There are two nginx based Ingress Controllers
  - https://github.com/kubernetes/ingress-nginx
  - https://github.com/nginxinc/kubernetes-ingress
- This guide is using the Ingress Controller project under NGINX Inc's [repo](https://github.com/nginxinc/kubernetes-ingress)

## Prerequisites
- Bare metal working kubernetes cluster

## Steps

#### NGINX Ingress Controller manifests

NGINX Ingress Controller manifests are available [here](https://github.com/nginxinc/kubernetes-ingress/tree/v1.5.8/deployments)

```
export MANIFESTS=https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v1.5.8/deployments
```

#### Create namespace, secrets, config maps and rbac

```
{
  kubectl create -f ${MANIFESTS}/common/ns-and-sa.yaml
  kubectl create -f ${MANIFESTS}/common/default-server-secret.yaml
  kubectl create -f ${MANIFESTS}/common/nginx-config.yaml
  kubectl create -f ${MANIFESTS}/rbac/rbac.yaml
}
```

#### Create Ingress Controller as a Deamonset

When Ingress Controller is deployed as a deamonset, ports 80 and 443 of the Ingress controller container are mapped
to the same ports of the node where the container is running. To access the Ingress controller, use those ports
and an IP address of any node of the cluster where the Ingress controller is running.

```
kubectl create -f ${MANIFESTS}/daemon-set/nginx-ingress.yaml
```

### Deploy the application and Ingress resource
#### Deploy the application
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: red
  labels:
    name: red
spec:
  volumes:
    - name: webdata
      emptyDir: {}
  initContainers:
    - name: web-content
      image: busybox
      volumeMounts:
        - name: webdata
          mountPath: "/webdata"
      command: ["/bin/sh", "-c", 'echo "RED" > /webdata/index.html']
  containers:
    - image: nginx
      name: nginx
      volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
EOF
```

#### Expose the application with ClusterIP Service
```
kubectl expose pod red --port 80 --name red-svc
```

#### Deploy Ingress resource

_Note: All annotations honered by NGINX Inc' Ingress Controller is defined [here](https://github.com/nginxinc/kubernetes-ingress/blob/v1.5.8/docs/configmap-and-annotations.md)_

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
  annotations:
    nginx.org/rewrites: |
      serviceName=red-svc rewrite=/;
spec:
  rules:
  - host: red.example.com
    http:
      paths:
      - backend:
          serviceName: red-svc
          servicePort: 80
  - host: example.com
    http:
      paths:
      - path: /red
        backend:
          serviceName: red-svc
          servicePort: 80

EOF
```

### Demo

For the demo purpose, get the IP address any of the worker node and hit on it like following. In practice a loadbalancer, with all the worker node's IP configured in the backend, should be used.

```
ubuntu@master01:~$ curl -D- <WORKER_NODE_IP> -H 'Host: red.example.com'
HTTP/1.1 200 OK
Server: nginx/1.17.6
Date: Wed, 18 Dec 2019 06:59:19 GMT
Content-Type: text/html
Content-Length: 4
Connection: keep-alive
Last-Modified: Wed, 18 Dec 2019 06:56:53 GMT
ETag: "5df9cdb5-4"
Accept-Ranges: bytes

RED
ubuntu@master01:~$ curl -D- <WORKER_NODE_IP>/red -H 'Host: example.com'
HTTP/1.1 200 OK
Server: nginx/1.17.6
Date: Wed, 18 Dec 2019 06:59:31 GMT
Content-Type: text/html
Content-Length: 4
Connection: keep-alive
Last-Modified: Wed, 18 Dec 2019 06:56:53 GMT
ETag: "5df9cdb5-4"
Accept-Ranges: bytes

RED
ubuntu@master01:~$ curl -D- <WORKER_NODE_IP>/test -H 'Host: example.com'
HTTP/1.1 404 Not Found
Server: nginx/1.17.6
Date: Wed, 18 Dec 2019 06:59:38 GMT
Content-Type: text/html
Content-Length: 153
Connection: keep-alive

<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.6</center>
</body>
</html>
```
