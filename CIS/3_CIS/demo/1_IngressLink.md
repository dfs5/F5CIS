    kubectl create -f bigip-login.yaml
    kubectl create serviceaccount bigip-ctlr -n kube-system
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/mdditt2000/k8s-bigip-ctlr/main/user_guides/ingresslink/nodeport/cis/ cis-crd-schema/customresourcedefinition.yaml -n kube-system
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

    https://cafe.example.com/coffee<script>

Don't forget to remove custom resource befor proceeding to the next.

    kubectl delete -f ingresslink.yaml -n nginx-ingress
