## Preparation
This lab is based on https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/ingresslink/nodeport/README.md

I recommend watching this Demo on [YouTube](https://www.youtube.com/watch?v=wi7vVZWHyxE) to better understand the idea of IngressLink using NodePort.

![architecture](https://github.com/dfs5/F5CIS/blob/master/CIS/3_CIS/diagram/2021-03-01_15-41-39%202.jpg)

On BIG-IP create a parition called 'k8s-01' 

Create the [Proxy Protocol](https://devcentral.f5.com/s/articles/How-to-persist-real-IP-into-Kubernetes) iRule on Bigip: \
Note: Proxy Protocol is only required for IngressLink

- Login to BigIp GUI
- On the Main tab, click Local Traffic > iRules.
- Click Create.
- In the Name field, type name as "Proxy_Protocol_iRule".
- In the Definition field, Copy the definition from "Proxy_Protocol_iRule" file. Click Finished.
proxy_protocol_iRule [repo](https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/IngressLink/Proxy_Protocol_iRule)

Ensure cafe application is running and the ingress is configured correctly.

    kubectl delete ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/1_tls_example.com.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/2_cafe-ingress-waf.yaml
    
In Browser verify NGINX IC configuration. You should see cafe.example.com with 2 upstreams to /tea and /coffee

    http://nodeIP:30003/dashboard.html

In browser verify that you can access the application directly via NGINX IC via Node Port: 30002

    https://cafe.example.com:30002/tea

## Next we will install CIS and access the same application via BIG-IP as external LB
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

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/rbac.yaml

Note: IngressLink requires CRDs. So we install F5 CRDs and the CRD for IngressLink.

    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/Install/customresourcedefinitions.yml
    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/IngressLink/ingresslink-customresourcedefinition.yaml

Note: Modify the arguments in the default CIS deployment to mach your environment.

##### args: 
#####   - "--bigip-username=$(BIGIP_USERNAME)"  <--- in this lab I use bigip-login.yaml to configure a secret 
#####   - "--bigip-password=$(BIGIP_PASSWORD)"  <--- that populates the content of these variabels
#####   - "--bigip-url=10.1.1.2"                <--- adjust to BIG-IP management IP
#####   - "--bigip-partition=k8s-01".           <--- musst match the BIG-IP partition name
#####   - "--namespace=nginx-ingress"           <--- add all namespaces IC should monitor, if your backend is KIC only that namespace is reqired
#####   - "--namespace=cafe"                    <--- example how to add additional namespaces 
#####   - "--pool-member-type=nodeport"         <--- using nodeport for easy integration; no overlay networks required 
#####   - "--log-level=DEBUG"                   <--- for test environaments, else use INFO
#####   - "--insecure=true"
#####   - "--log-as3-response=true"
#####   - "--custom-resource-mode=true"         <--- IngressLink reqires using CRD
    
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cis-deployment-nodeport.yaml

Verify CIS pod is running.

    $ kubectl get pod -n kube-system | grep k8s
    k8s-bigip-ctlr-deployment-65855bfdb6-826b4   1/1     Running   0          47s


## Monitor 
Monitor AS3 calls generated on CIS:

    kubectl logs -f k8s-bigip-ctlr-deployment-pod-id -n kube-system | grep --color=auto -i '\[as3'

Monitor NGINX IC logs:

    kubectl logs -f nginx-ingress-pod-id -n nginx-ingress
    
Monitor NGINX IC traffic:

    http://NodeIP:30003/dashboard.html

## Next we will deploy the following 3 Options:
![architecture](https://github.com/dfs5/F5CIS/blob/master/CIS/3_CIS/diagram/Screenshot%202021-05-17%20at%2017.18.43.png)

## Deploy IngressLink resource for connectivity to BIG-IP
Note: Adjust the VIP IP address as per your environment. That's it!\
IngressLink has a specific Health Check on port 8081 included for NGINX IC instances.
Also IngressLink uses Proxy Mode to advertise client IP to the NGINX instance.


    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/nginx-nodeport-health.yaml
    |---> opening port 8081 for Health Check; adding label for IngressLink selector
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/configmap_proxy-mode.yaml
    kubectl apply -f ingresslink.yaml
    |---> applying configuration to BIG-IP; monitor your CIS AS3 communication
    
See CR configuration in BIG-IP UI!!!

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is running:

    https://cafe.example.com/coffee<script>

Note: Don't forget to remove custom resource befor proceeding to the next.\
Note: Show that configuration has been removed from the BIG-IP!!!
    
    kubectl delete ingresslink il-cluster-vip -n nginx-ingress

## Deploy TransportServer resource for connectivity to BIG-IP
As of today similar use case as with IngressLink but more flexible. E.g. you can define any port. (see: [IngressLink CRD vs TransportServer CRD](https://devcentral.f5.com/s/articles/My-first-deployment-of-IngressLink))\
Note: First remove Proxy mode from NGINX IC.

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ts_tcp.yaml

See CR configuration in BIG-IP UI!!!

Access cafe-app from browser:

    https://cafe.example.com:8443/coffee
    
Show that NAP is still running:

    https://cafe.example.com:8443/coffee?dfs=<script>
    
Note: Delete custom resource when you are finished and check configuration is removed from BIG-IP. 

    kubectl delete transportserver transport-server -n nginx-ingress

## Deploy VirtualServer resource for connectivity to BIG-IP
As of today this is the only option to use L7 services on BIG-IP with CRDs. With that you can terminate SSL and leverage BIG-IP WAF policies for traffic inspection.

In this lab we will disable tls termination in NGINX IC and move it to BIG-IP. On the BIG-IP we will have a standard HTTP VIP frontending our K8S cluster. NGINX IC is still doing NAP.

    kubectl delete Ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cafe-ingress-waf_noTLS.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vsp_nginx-cafe-terminate-tls.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vs_nginx-cafe.yaml

See CR configuration in BIG-IP UI!!!

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is still running:

    https://cafe.example.com/coffee?dfs=<script>

Note: Delete custom resource when you are finished and check configuration is removed from BIG-IP.
    
    kubectl delete -f vs_nginx-cafe.yaml
    kubectl delete -f vsp_nginx-cafe-terminate-tls.yaml

## Delete CIS lab

    kubectl delete deployment k8s-bigip-ctlr-deployment -n kube-system
    kubectl delete CustomResourceDefinition virtualservers.cis.f5.com ingresslinks.cis.f5.com
    kubectl delete serviceaccount bigip-ctlr
    kubectl delete secret bigip-login -n kube-system
    kubectl delete ClusterRoleBinding bigip-ctlr-clusterrole-binding
    kubectl delete ClusterRole bigip-ctlr-clusterrole
