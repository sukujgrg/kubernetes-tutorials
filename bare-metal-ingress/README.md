Ingress controller on Bare metal
---

### What is required for this example?
- a kubernetes cluster
- a load balancer (eg: HaProxy)
- Ingress Controller (eg: kubernetes-ingress from nginx-inc) / it's manifests

### Steps

#### ns, secrets, config maps and rbac
```
export BASE_URL_INGRESS_CONTROLLER_YAMLS=https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v1.5.8

kubectl create -f ${BASE_URL_INGRESS_CONTROLLER_YAMLS}/deployments/common/ns-and-sa.yaml
kubectl create -f ${BASE_URL_INGRESS_CONTROLLER_YAMLS}/deployments/common/default-server-secret.yaml
kubectl create -f ${BASE_URL_INGRESS_CONTROLLER_YAMLS}/deployments/common/nginx-config.yaml
kubectl create -f ${BASE_URL_INGRESS_CONTROLLER_YAMLS}/deployments/rbac/rbac.yaml
```

#### deploy ingress controller as a deamonset
When ingress controller is deployed as a deamonset, ports 80 and 443 of the Ingress controller container are mapped
to the same ports of the node where the container is running. To access the Ingress controller, use those ports
and an IP address of any node of the cluster where the Ingress controller is running.

```
export BASE_URL_INGRESS_CONTROLLER_YAMLS=https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v1.5.8

kubectl create -f ${BASE_URL_INGRESS_CONTROLLER_YAMLS}/deployments/daemon-set/nginx-ingress.yaml
```


#### deploy application and expose it

```
kubectl create -f resources/ingress-demo-web-one.yaml
kubectl expose pod nginx-one --port 80 --name nginx-one
```

#### deploy ingress resource

```
kubectl apply -f resources/ingress-resource-1.yaml
```

