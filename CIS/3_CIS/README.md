## Preparation
This lab is based on https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ingresslink/nodeport/README.md

I recommend watching this Demo on [YouTube](https://www.youtube.com/watch?v=wi7vVZWHyxE) to better understand the idea of IngressLink using NodePort.

![architecture](https://github.com/dfs5/F5CIS/blob/master/CIS/3_CIS/diagram/2021-03-01_15-41-39%202.jpg)

On BIG-IP create a parition called 'k8s-01' 

Create the Proxy Protocol iRule on Bigip: \
Note: Proxy Protocol is only required for IngressLink

- Login to BigIp GUI
- On the Main tab, click Local Traffic > iRules.
- Click Create.
- In the Name field, type name as "Proxy_Protocol_iRule".
- In the Definition field, Copy the definition from "Proxy_Protocol_iRule" file. Click Finished.
proxy_protocol_iRule [repo](https://github.com/mdditt2000/kubernetes-1-19/blob/master/cis%202.3/github/damian/ingresslink/big-ip/proxy-protocal/irule)

## Installing CIS Manually
(https://clouddocs.f5.com/containers/latest/userguide/kubernetes/)

Note: Create 'bigip-login.yaml' with your admin password.

    vi bigip-login.yaml

    apiVersion: v1
    kind: Secret
    metadata:
      name: bigip-login
      namespace: kube-system
    type: kubernetes.io/basic-auth
    stringData:
      username: admin
      password: xxxxx
    
    kubectl create -f bigip-login.yaml
    kubectl create serviceaccount bigip-ctlr -n kube-system

Note: RBAC should be changed as per your cluster requirements: 'controller_namespace' 'secret-containing-bigip-login'

    kubectl apply -f rbac.yaml

Note: IngressLink requires CRDs. So we install F5 CRDs.

    kubectl apply -f https://raw.githubusercontent.com/mdditt2000/k8s-bigip-ctlr/main/user_guides/ingresslink/nodeport/cis/cis-crd-schema/customresourcedefinition.yaml -n kube-system

Note: Modify the default CIS deployment.

##### args: 
#####   - "--bigip-username=$(BIGIP_USERNAME)"
#####   - "--bigip-password=$(BIGIP_PASSWORD)"
#####   - "--bigip-url=10.1.1.2"                <--- adjust IP
#####   - "--bigip-partition=k8s-01"
#####   - "--namespace=nginx-ingress"           <--- add all namespaces IC should monitor
#####   - "--namespace=cafe"                    <--- add all namespaces IC should monitor 
#####   - "--pool-member-type=nodeport"         <--- using nodeport for easy integration; no overlay networks required 
#####   - "--log-level=DEBUG"
#####   - "--insecure=true"
#####   - "--log-as3-response=true"
#####   - "--custom-resource-mode=true"         <--- using CRD for configuration simplicity
    
    kubectl apply -f cis-deployment-nodeport.yaml

Verify CIS pod is running.

    $ kubectl get pod -n kube-system | grep k8s
    k8s-bigip-ctlr-deployment-65855bfdb6-826b4   1/1     Running   0          47s


## Modify nginx configuration

Note: Use Proxy mode on NGINX IC for IngressLink

    kubectl apply -f https://raw.githubusercontent.com/mdditt2000/k8s-bigip-ctlr/main/user_guides/ingresslink/nodeport/nginx-config/nginx-config.yaml


## Monitor 
Monitor AS3 calls on CIS:

    kubectl logs -f k8s-bigip-ctlr-deployment-pod-id -n kube-system | grep --color=auto -i '\[as3'

Monitor NGINX IC traffic:

    kubectl logs -f nginx-ingress-pod-id -n nginx-ingress

## Deploy IngressLink resource for connectivity to BIG-IP
Only IP address needs to be configured. That's it!\
Also IngressLink has a specific Health Check on port 8081 included for NGINX IC instances.

    kubectl apply -f nginx-nodeport-health.yaml    <--- opening port 8081 for Health Check; adding label for IngressLink selector
    kubectl apply -f ingresslink.yaml              <--- applying configuration to BIG-IP; monitor your CIS AS3 communication
    
Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is running:

    https://cafe.example.com/coffee<script>

Note: Don't forget to remove custom resource befor proceeding to the next.
    
    kubectl delete -f ingresslink.yaml

## Deploy TransportServer resource for connectivity to BIG-IP
As of today similar use case as with IngressLink but more flexible. E.g. you can define any port. (see: [IngressLink CRD vs TransportServer CRD](https://devcentral.f5.com/s/articles/My-first-deployment-of-IngressLink))\
Note: First remove Proxy mode from NGINX IC.

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml

    kubectl apply -f ts_tcp.yaml

Access cafe-app from browser:

    https://cafe.example.com:8443/coffee
    
Verify NAP is running:

    https://cafe.example.com:8443/coffee<script>
    
Note: Delete custom resource when you are finished. 

    kubectl delete -f ts_tcp.yaml

## Deploy VirtualServer resource for connectivity to BIG-IP
As of today this is the only option to use L7 services on BIG-IP with CRDs. With that you can terminate SSL and leverage BIG-IP WAF policies for traffic inspection.

In this lab we will disable tls termination in NGINX IC and move it to BIG-IP. On the BIG-IP we will have a standard HTTP VIP frontending our K8S cluster. NGINX IC is still doing NAP.

    kubectl delete Ingress cafe-ingress -n cafe
    kubectl apply -f 3b_cafe-ingress-waf_noTLS.yaml
    kubectl apply -f vsp_nginx-cafe-terminate-tls.yaml
    kubectl apply -f vs_nginx-cafe.yaml

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is running:

    https://cafe.example.com/coffee<script>

Note: Delete custom resource when you are finished.
    
    kubectl delete -f vs_nginx-cafe.yaml
    kubectl delete -f vsp_nginx-cafe-terminate-tls.yaml

## Delete all

    kubectl delete -f cis-deployment-nodeport.yaml
    kubectl delete CustomResourceDefinition virtualservers.cis.f5.com -n kube-system
    kubectl delete serviceaccount bigip-ctlr -n kube-system
    kubectl delete -f bigip-login.yaml
    kubectl delete -f rbac.yaml