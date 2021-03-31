    !/bin/bash
    
    #create kubernetes cis container, authentication and RBAC
    kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
    kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin   --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
    # kubectl create secret generic bigip-login -n kube-system  --from-literal=username=admin --from-literal=password=*****
    kubectl create -f bigip-login-secret.yaml
    kubectl create -f f5-cis-deployment_vs.yaml
    kubectl create -f crds.yaml
