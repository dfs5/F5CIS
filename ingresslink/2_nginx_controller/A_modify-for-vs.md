
kubectl delete -f 2_nginx_controller/nginx-config.yaml 

kubectl create -f 2_nginx_controller/nginx-config-noproxy-protocol.yaml 

kubectl delete -f 2_nginx_controller/nginx-ingress.yaml 

kubectl create -f 2_nginx_controller/nginx-ingress-no-ingresslink.yaml

#comments:
#'data:' content removed in 'nginx-config-noproxy-protocol.yaml'
#'- -ingresslink=nginx-ingress' removed in 'nginx-ingress-no-ingresslink.yaml'