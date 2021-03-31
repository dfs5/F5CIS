
    kubectl delete ns cafe

    kubectl delete -f 3_bigip_ingresslink_resource/vs-securework_tea_coffee.yaml
    kubectl delete -f 3_bigip_ingresslink_resource/terminate-tls.yaml
    
    kubectl delete -f 2_nginx_controller/ns-and-sa.yaml
    kubectl delete -f 2_nginx_controller/rbac/rbac.yaml
    kubectl delete -f 2_nginx_controller/rbac/ap-rbac.yaml
    kubectl delete -f 2_nginx_controller/ingress-class.yaml
    
    kubectl delete -f 2_nginx_controller/crds/k8s.nginx.org_virtualservers.yaml
    kubectl delete -f 2_nginx_controller/crds/k8s.nginx.org_virtualserverroutes.yaml
    kubectl delete -f 2_nginx_controller/crds/k8s.nginx.org_policies.yaml
    
    kubectl delete deployment k8s-bigip-ctlr-deployment -n kube-system
    kubectl delete clusterrolebinding k8s-bigip-ctlr-clusteradmin
    kubectl delete serviceaccount k8s-bigip-ctlr -n kube-system
    kubectl delete -f 1_cis_controller/bigip-login-secret.yaml
    kubectl delete -f 1_cis_controller/crds.yaml

