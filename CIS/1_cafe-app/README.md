## Install Cafe App
    kubectl apply -f 1_cafe-ns.yaml
    kubectl apply -f 2_cafe-app.yaml
## Deploy NGINX IC first befor proceeding (2_nginx-ic-plus)
    kubectl apply -f 3a_tls_example.com.yaml
    kubectl apply -f 3b_cafe-ingress-waf.yaml
## Verify Ingress configuration
    kubectl exec nginx-ingress-pod-id -n nginx-ingress -- nginx -T
## Uninstall
    kubectl delete namespace cafe