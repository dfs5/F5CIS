## Manage node labels

**Problem:** When using nodeport by default all nodes from the cluster will be added to the pool. In most cases you only want to add the worker nodes and exclude master nodes. To exclude master nodes using the label node-role.kubernetes.io/node parameter

**Solution:** Use the label node to create a new label for the node. This works in conjunction with the node-label-selector configured in CIS to only add nodes to the pool with the associated . In this quick start guide CIS will only add the nodes with label worker to the pool. Excluding the master node from the pool

```
kubectl label nodes k8s-1-18-node1.example.com f5role=worker
kubectl label nodes k8s-1-18-node2.example.com f5role=worker
```

Show the node labels

```
kubectl get nodes
NAME                         STATUS   ROLES    AGE     VERSION
k8s-1-18-master.example.com   Ready    master   7d19h   v1.18.0
k8s-1-18-node1.example.com    Ready    f5role   7d19h   v1.18.0
k8s-1-18-node2.example.com    Ready    f5role   7d19h   v1.18.0
args:
- "--bigip-username=$(BIGIP_USERNAME)"
- "--bigip-password=$(BIGIP_PASSWORD)"
- "--bigip-url=192.168.200.92"
- "--bigip-partition=k8s"
- "--namespace=default"
- "--pool-member-type=nodeport"
- "--log-level=DEBUG"
- "--insecure=true"
- "--manage-ingress=false"
- "--manage-routes=false"
- "--manage-configmaps=true"
- "--agent=as3"
- "--as3-validation=true"
- "--node-label-selector=f5role=worker" - set the node label
```

## No label created

```
[kube@k8s-1-19-master crd-examples]$ kubectl get nodes
NAME                               STATUS   ROLES    AGE   VERSION
k8s-1-19-master.example.com   Ready    master   29d   v1.19.0
k8s-1-19-node1.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node2.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node3.example.com    Ready    <none>   29d   v1.19.0
k8s-1-19-node4.example.com    Ready    <none>   29d   v1.19.0
```
Create the label
```
kubectl label nodes k8s-1-19-node1.example.com f5role=worker
kubectl label nodes k8s-1-19-node2.example.com f5role=worker
kubectl label nodes k8s-1-19-node3.example.com f5role=worker
kubectl label nodes k8s-1-19-node4.example.com f5role=worker
```

Below you can see the label created **f5role=worker** for nodes 1 - 4

```
[kube@k8s-1-19-master 1531]$ kubectl get nodes --show-labels
NAME                               STATUS   ROLES    AGE   VERSION   LABELS
k8s-1-19-master.example.com   Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-master.example.com,kubernetes.io/os=linux
k8s-1-19-node1.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node1.example.com,kubernetes.io/os=linux
k8s-1-19-node2.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node2.example.com,kubernetes.io/os=linux
k8s-1-19-node3.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node3.example.com,kubernetes.io/os=linux
k8s-1-19-node4.example.com    Ready    <none>   30d   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,f5role=worker,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-1-19-node4.example.com,kubernetes.io/os=linux
```

## Example of NodeLabel in VS definition
```
apiVersion: "cis.f5.com/v1"
kind: VirtualServer
metadata:
  name: f5-demo
  labels:
    f5cr: "true"
spec:
  virtualServerAddress: "10.192.75.108"
  host: mysite.f5demo.com
  pools:
  - monitor:
      interval: 20
      recv: ""
      send: /
      timeout: 10
      type: http
    path: /
    service: f5-demo
    servicePort: 80
    nodeMemberLabel: f5role=worker
```