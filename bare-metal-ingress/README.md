Ingress controller on Bare metal
---

## What is required for this example?
- a kubernetes cluster
- a load balancer (eg: HaProxy)
- Ingress Controller (eg: kubernetes-ingress from nginx-inc) / it's manifests

## Steps

### Install Nginx Controller

source: [nginxinc/kubernetes](https://github.com/nginxinc/kubernetes-ingress/blob/v1.5.8/docs/installation.md)
nginx controller version : v1.5.8

#### Create namespace, secrets, config maps and rbac

```
export YAML_RESOURCES=https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v1.5.8/deployments

kubectl create -f ${YAML_RESOURCES}/common/ns-and-sa.yaml
kubectl create -f ${YAML_RESOURCES}/common/default-server-secret.yaml
kubectl create -f ${YAML_RESOURCES}/common/nginx-config.yaml
kubectl create -f ${YAML_RESOURCES}/rbac/rbac.yaml
```

#### Create ingress controller as a Deamonset

When ingress controller is deployed as a deamonset, ports 80 and 443 of the Ingress controller container are mapped
to the same ports of the node where the container is running. To access the Ingress controller, use those ports
and an IP address of any node of the cluster where the Ingress controller is running.

```
kubectl create -f ${YAML_RESOURCES}/daemon-set/nginx-ingress.yaml
```


### Deploy the application and Ingress resource

#### Deploy the application and expose it (ClusterIP)

```
kubectl create -f resources/ingress-demo-web-one.yaml
kubectl expose pod nginx-one --port 80 --name nginx-one
```

#### Deploy ingress resource

```
kubectl apply -f resources/ingress-resource-1.yaml
```

