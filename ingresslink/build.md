
    
    kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
    kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin   --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
    kubectl create -f 1_cis_controller/crds.yaml
    kubectl create -f 1_cis_controller/bigip-login-secret.yaml
    kubectl create -f 1_cis_controller/f5-cis-deployment.yaml

    kubectl create -f 2_nginx_controller/crds/k8s.nginx.org_virtualservers.yaml
    kubectl create -f 2_nginx_controller/crds/k8s.nginx.org_virtualserverroutes.yaml
    kubectl create -f 2_nginx_controller/crds/k8s.nginx.org_policies.yaml
    kubectl create -f 2_nginx_controller/crds/appprotect.f5.com_aplogconfs.yaml
    kubectl create -f 2_nginx_controller/crds/appprotect.f5.com_appolicies.yaml
    kubectl create -f 2_nginx_controller/crds/appprotect.f5.com_apusersigs.yaml

    kubectl create -f 2_nginx_controller/ns-and-sa.yaml
    kubectl create -f 2_nginx_controller/rbac/rbac.yaml
    kubectl create -f 2_nginx_controller/rbac/ap-rbac.yaml
    kubectl create -f 2_nginx_controller/ingress-class.yaml
    kubectl create -f 2_nginx_controller/default-server-secret.yaml
    kubectl create -f 2_nginx_controller/nginx-config.yaml
    kubectl create -f 2_nginx_controller/nginx-ingress.yaml
    kubectl create -f 2_nginx_controller/nginx-service.yaml

    kubectl create -f 3_bigip_ingresslink_resource/vs-ingresslink.yaml

    kubectl create -f 4_cafe-app/cafe-ns.yaml
    kubectl create -f 4_cafe-app/cafe-secret.yaml
    kubectl create -f 4_cafe-app/coffee.yaml
    kubectl create -f 4_cafe-app/cafe-ingress.yaml











