Ensure cafe application is running. 

    kubectl get pod,svc -n cafe

In Browser verify NGINX IC configuration. You shouldn't see cafe.example.com yet.

    http://AnyNodeIP:30003/dashboard.html  <-- AnyNode only if dashboard access allowed from 0.0.0.0/0 

Deploy the ingress for cafe.

    kubectl delete ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/1_tls_example.com.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/2_cafe-ingress-waf.yaml
    kubectl get ing -n cafe
    
In Browser verify NGINX IC configuration. You should now see cafe.example.com with 2 upstreams to /tea and /coffee

    http://AnyNodeIP:30003/dashboard.html


    kubectl create -f bigip-login.yaml
    kubectl create serviceaccount bigip-ctlr -n kube-system
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/Install/customresourcedefinitions.yml
    kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/crd/IngressLink/ingresslink-customresourcedefinition.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/configmap_proxy-mode.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cis-deployment-nodeport.yaml

Verify CIS pod is running.

    kubectl get pod -n kube-system | grep k8s

In a separate ssh session monitor AS3 calls generated on CIS

    kubectl logs -f k8s-bigip-ctlr-deployment-pod-id -n kube-system | grep --color=auto -i '\[as3'

In Browser monitor NGINX IC traffic

    http://NodeIP:30003/dashboard.html

Deploy IngressLink resource for connectivity to BIG-IP

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/nginx-nodeport-health.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ingresslink.yaml

Show CR configuration in BIG-IP UI!!!

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is running:

    https://cafe.example.com/coffee?dfs=<script>

Don't forget to remove custom resource befor proceeding to the next.\
Note: Show that configuration has been removed from the BIG-IP!!!

    kubectl delete ingresslink il-cluster-vip -n nginx-ingress
