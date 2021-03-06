## Install Cafe App
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/1_cafe-ns.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/2_cafe-app.yaml
## Deploy [NGINX IC](https://github.com/dfs5/F5CIS/tree/master/CIS/2_nginx-ic-plus) first befor proceeding
## Deploy WAF-policy, ssl certificate and ingress configuration for cafe app
Adjust the syslog server IP in '2_cafe-ingress-waf.yaml' to your current service clusterIP befor deploying

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/waf-policy.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/1_tls_example.com.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/1_cafe-app/kind_ingress/2_cafe-ingress-waf.yaml
## Verify Ingress configuration
Verify Ingress configuration on NGINX dashboard (!!! nginx-status-allow-cidrs=10.24.0.0/16 !!!)

    In browser: http://nodeIP:30003/dashboard.html#
Optinal verify NGINX config file

    kubectl exec nginx-ingress-pod-id -n nginx-ingress -- nginx -T
Access application from browser:

    https://cafe.example.com:30002/coffee
## Appendix: Uninstall cafe application
    kubectl delete namespace cafe
