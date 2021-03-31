

    kubectl delete -f 2_nginx_controller/nginx-config-noproxy-protocol.yaml 
    kubectl create -f 2_nginx_controller/nginx-config.yaml 

    kubectl delete -f 2_nginx_controller/nginx-ingress-no-ingresslink.yaml
    kubectl create -f 2_nginx_controller/nginx-ingress.yaml 


comments:
- 'data:' content added in 'nginx-config.yaml'
- '- -ingresslink=nginx-ingress' added in 'nginx-ingress.yaml'