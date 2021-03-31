# F5CIS

## Requirements to run the lab
- BIGIP HW or VE
- Kubernetes Cluster >=1.18
- All services are running in NodePort mode so no overlay networks like flannel or calico are required
- Minimum version to use IngressLink:

| CIS | BIGIP | NGINX+ IC | AS3 |
| ------ | ------ | ------ | ------ |
| 2.4+ | v13.1+ | 1.10+ | 3.18+ | 

## Navigation
ingresslink - start here to demomstrate the beta version

virtualserver - start here to demonstrate CRD based BIGIP configuration as external IC

verify-clean-start.md - befor you start verify that all the resources being created have no conflicts with your current cluster deployments