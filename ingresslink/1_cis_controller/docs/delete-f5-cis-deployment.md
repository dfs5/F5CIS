    #!/bin/bash

    #delete kubernetes cis container, authentication and RBAC
    
    kubectl delete deployment k8s-bigip-ctlr-deployment -n kube-system
    kubectl delete clusterrolebinding k8s-bigip-ctlr-clusteradmin
    kubectl delete serviceaccount k8s-bigip-ctlr -n kube-system
    kubectl delete -f crds.yaml
    # kubectl delete secret bigip-login -n kube-system
    kubectl delete -f bigip-login-secret.yaml