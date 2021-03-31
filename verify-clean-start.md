
# All output has to be empty:

    kubectl get ns |grep cafe
    kubectl get ns |grep arcadia
    kubectl get ns |grep nginx-ingress
    kubectl get pod -n kube-system |grep bigip
    kubectl get pod -n nginx-ingress
    kubectl get pod -n cafe
    kubectl get pod -n arcadia
    kubectl get ingress -A
    kubectl get svc -A |grep coffee-svc
    kubectl get svc -A |grep tea-svc
    kubectl get svc -A |grep nginx-ingress
    kubectl get svc -n arcadia
    kubectl get secret -n kube-system |grep bigip-login