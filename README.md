# F5CIS

This lab demostrates different options to connect a K8S cluster to a BIG-IP load balancer to allow access to cluster applications from the internet. It leverages BIG-IP as transport service and NGINX+ as Ingress Controller with L7 security modul NAP.    

## Requirements to run the lab
- BIGIP HW or VE
- Kubernetes Cluster >=1.18
- All services running in NodePort mode; no overlay networks required
- Minimum version to use IngressLink:

| CIS | BIGIP | NGINX+ IC | AS3 |
| ------ | ------ | ------ | ------ |
| 2.4+ | v13.1+ | 1.10+ | 3.18+ | 

## Navigation
CIS - start here to demomstrate custome resource deployments with BIG-IP + NGINX IC including NGINX App Protect (NAP)

PrivateRegistry - guidance to build a local registry to host your NGINX+ build for cluster deployments