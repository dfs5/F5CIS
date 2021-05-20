Delete CIS lab

    kubectl delete deployment k8s-bigip-ctlr-deployment -n kube-system
    kubectl delete CustomResourceDefinition virtualservers.cis.f5.com ingresslinks.cis.f5.com
    kubectl delete serviceaccount bigip-ctlr
    kubectl delete secret bigip-login -n kube-system
    kubectl delete ClusterRoleBinding bigip-ctlr-clusterrole-binding
    kubectl delete ClusterRole bigip-ctlr-clusterrole