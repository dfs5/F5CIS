## Preparation
This lab is based on https://github.com/F5Networks/k8s-bigip-ctlr/tree/master/docs/config_examples/crd/IngressLink

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
proxy_protocol_iRule [repo](https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/customResource/IngressLink/Proxy_Protocol_iRule)

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

Note: RBAC should be changed as per your cluster requirements: 'controller_namespace' 'secret-containing-bigip-login'. The second rbac is for the IPAM controller for later use.

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ipam/f5-ipam-rbac.yaml

Note: IngressLink is based on CRDs. So we install F5 CRDs for IngressLink, IPAM and other F5 CRDs for later use cases.


Limitations when CIS deployed in CRD mode:
- CIS does not watch for Ingress/Routes/ConfigMaps when deployed in CRD Mode.
- CIS does not support the combination of CRDs with any of Ingress/Routes and ConfigMaps.

Limitations when CIS deployed in CRD mode:
- CIS does not watch for Ingress/Routes/ConfigMaps when deployed in CRD Mode.
- CIS does not support the combination of CRDs with any of Ingress/Routes and ConfigMaps.

    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/Install/customresourcedefinitions.yml
    
    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/IngressLink/ingresslink-customresourcedefinition.yaml
    
    kubectl apply -f https://raw.githubusercontent.com/F5Networks/f5-ipam-controller/main/docs/_static/schemas/ipam_schema.yaml
    
[IPAM Integration](https://github.com/F5Networks/f5-ipam-controller): We deploy IPAM controller for IP management. With that CRD configuration becomes even easier as IP addresses are automatically assigned from a predefined range. Integration with Infoblox is possible. Change the IP ranges to fit your requirements.

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ipam/f5-ipam-deployment_default.yaml

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
#####   - "--ipam=true"                         <--- Integration with IPAM controller

    
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cis-deployment-nodeport.yaml

Verify CIS and IPAM pods are running.

    $ kubectl get pod -n kube-system | grep k8s
    k8s-bigip-ctlr-deployment-65855bfdb6-826b4   1/1     Running   0          47s
    
    $ kubectl get pod -n kube-system | grep ipam
    f5-ipam-controller-5cfb94bb8b-7dm8l          1/1     Running   0          117s


## Monitor 
Monitor AS3 calls generated by CIS:

    kubectl logs -f k8s-bigip-ctlr-deployment-pod-id -n kube-system | grep --color=auto -i '\[as3'

Monitor NGINX IC logs:

    kubectl logs -f nginx-ingress-pod-id -n nginx-ingress
    
Monitor IPAM logs:

    kubectl logs -f f5-ipam-controller-pod-id -n kube-system
    
## Next we will deploy the following 3 Options:
![architecture](https://github.com/dfs5/F5CIS/blob/master/CIS/3_CIS/diagram/Screenshot%202021-08-13%20at%2011.00.02.png)

## Deploy IngressLink resource for connectivity to BIG-IP
Note: Adjust the VIP IP address as per your environment. That's it!\
IngressLink has a specific Health Check on port 8081 included for NGINX IC instances.
Also IngressLink uses Proxy Mode to advertise client IP to the NGINX instance.


    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/nginx-nodeport-health.yaml
    |---> opening port 8081 for Health Check; adding label for IngressLink selector
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/configmap_proxy-mode.yaml
    kubectl apply -f ingresslink.yaml
    |---> applying configuration to BIG-IP; monitor your CIS AS3 communication
    
Verify VirtualServer configuration in BIG-IP UI!!!
- 2x VIPs (80/443) with tcp profile only, no SSL, no http
- pool assigned via Default Pool configuration, no LTM traffic policies
- predefined iRule for Proxy_Protocol attached
- healthcheck based on NGINX ready state: GET /nginx-ready HTTP/1.1\r\n

You can optional access NGINX Readines port from browser: http://AnyNodeIP:Port/nginx-ready 

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is running:

    https://cafe.example.com/coffee<script>

This finalize the first Use Case.\
Note: Don't forget to remove CR with the command below befor proceeding to the next use case.\
Note: Monitor AS3 log to see sucessfull API declaration. Check in the BIG-IP UI that VIP configuration has been removed!!!
    
    kubectl delete ingresslink il-cluster-vip -n nginx-ingress

## Deploy TransportServer resource for connectivity to BIG-IP
Similar use case as with IngressLink but more flexible. E.g. you can define protocol tcp or udp and any port. (see: [IngressLink CRD vs TransportServer CRD](https://devcentral.f5.com/s/articles/My-first-deployment-of-IngressLink))\
Here we apply a VIP listening on port 8443 with a tcp profile and a simple tcp monitor and attach the same Proxy_Protocol_iRule as with IngressLink. We are using the nginx-nodeport-health.yaml and configmap_proxy-mode.yaml from prevoius configuration.

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ts_tcp.yaml

Verify VirtualServer configuration in BIG-IP UI!!!

Access cafe-app from browser:

    https://cafe.example.com:8443/coffee
    
Verify NAP is running:

    https://cafe.example.com:8443/coffee<script>
    
    
IPAM integration: Now apply 2 additional servers with IP addresses being provided automatically by IPAM.
![architecture](https://github.com/dfs5/F5CIS/blob/master/CIS/3_CIS/diagram/Screenshot%202021-08-13%20at%2011.02.22.png)

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ts_tcp_ipam.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ts_tcp_ipam2.yaml

Verify VirtualServer configuration in BIG-IP UI!!! You should see now 3 VS in total.

Verify IPAM configuration.\
Note: Due to a possible [BUG](https://github.com/F5Networks/k8s-bigip-ctlr/issues/1916) the IP address assigned from IPAM static range is not shown here.

    kubectl get ts -n nginx-ingress
    NAME                     VIRTUALSERVERADDRESS   VIRTUALSERVERPORT   POOL            POOLPORT   AGE
    transport-server         10.24.1.22             8443                nginx-ingress   443        10m
    transport-server-ipam                           8443                nginx-ingress   443        35s
    transport-server-ipam2                          8443                nginx-ingress   443        34s

This finalize the second Use Case.\
Note: Don't forget to remove custom resource with the command below befor proceeding to the next use case.\
Note: Monitor AS3 log to see sucessfull API declaration. Check in the BIG-IP UI that VIP configuration has been removed!!! 

    kubectl delete transportserver transport-server -n nginx-ingress
    kubectl delete transportserver transport-server-ipam -n nginx-ingress
    kubectl delete transportserver transport-server-ipam2 -n nginx-ingress

## Deploy VirtualServer resource for connectivity to BIG-IP
As of today this is the only option to use L7 services on BIG-IP with CRDs. With that you can terminate SSL and leverage BIG-IP WAF policies for traffic inspection.

In this lab we will disable tls termination in NGINX IC and move it to BIG-IP. On the BIG-IP we will have a standard HTTP VIP frontending our K8S cluster. NGINX IC is still doing NAP.\
Note: "Proxy_Protocol_iRule" is not used in that configuration so we apply an empty ConfigMap to the NGINX IC instance.\
Note: Health check will be done on application so we remove Readines Port 8081 and use activate NGINX Dashboard for demo purposes.

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/nodeport_dashboard.yaml
    kubectl delete Ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cafe-ingress-waf_noTLS.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vsp_nginx-cafe-terminate-tls.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vs_nginx-cafe.yaml

Verify VirtualServer configuration in BIG-IP UI!!!
- 2x standard VIPs (80 for redirect/443 for app workloads), http profile assigned
- pool assigned via LTM traffic policies matching on "HTTP header" and "HTTP URI" 
- extended healthcheck based on application state: GET /coffee HTTP/1.1\r\nHost: cafe.example.com\r\nConnection: Close\r\n\r\n

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is still running:\
Note: We have WAF still running on NGINX but now it could be run on the frontend BIG-IP as well!!!

    https://cafe.example.com/coffee?dfs=<script>

IPAM Integration: Now apply 2 additional servers with IP addresses being provided automatically by IPAM.

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vs_nginx-cafe_ipam.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vs_nginx-cafe_ipam2.yaml

Verify IPAM configuration.\
Note: Due to a possible [BUG](https://github.com/F5Networks/k8s-bigip-ctlr/issues/1916) the IP address assigned from IPAM static range is not shown here.

    kubectl get vs -n nginx-ingress
    NAME                  HOST                TLSPROFILENAME   HTTPTRAFFIC   IPADDRESS    IPAMLABEL   IPAMVSADDRESS   AGE
    vs-nginx-cafe         cafe.example.com    terminate-tls    redirect      10.24.1.22                               68s
    vs-nginx-cafe-ipam    cafe2.example.com   terminate-tls                               Dev                         52s
    vs-nginx-cafe-ipam2   cafe3.example.com   terminate-tls    redirect                   Dev                         50s

This finalize the third and last Use Case.\
Note: Don't forget to remove custom resource with the command below.\
Note: Monitor AS3 log to see sucessfull API declaration. Check in the BIG-IP UI that VIP configuration has been removed!!! 
    
    kubectl delete vs -n nginx-ingress vs-nginx-cafe
    kubectl delete vs -n nginx-ingress vs-nginx-cafe-ipam
    kubectl delete vs -n nginx-ingress vs-nginx-cafe-ipam2
    kubectl delete tls -n nginx-ingress terminate-tls

## Delete CIS deployment when you finished the lab.

    kubectl delete deployment k8s-bigip-ctlr-deployment -n kube-system
    kubectl delete CustomResourceDefinition virtualservers.cis.f5.com ingresslinks.cis.f5.com
    kubectl delete serviceaccount bigip-ctlr
    kubectl delete secret bigip-login -n kube-system
    kubectl delete ClusterRoleBinding bigip-ctlr-clusterrole-binding
    kubectl delete ClusterRole bigip-ctlr-clusterrole
    
## Restore access via NGINX IC to reset the lab for the next demo.

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/nodeport_dashboard.yaml
    kubectl delete ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/1_tls_example.com.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/2_cafe-ingress-waf.yaml

