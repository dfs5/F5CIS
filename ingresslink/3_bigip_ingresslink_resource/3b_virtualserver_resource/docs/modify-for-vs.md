
    kubectl delete -f vs-ingresslink.yaml
    
    kubectl create -f 3b_virtualserver_resource/terminate-tls.yaml
    kubectl create -f 3b_virtualserver_resource/vs-securework_tea_coffee.yaml 
    
    kubectl create -f 3b_virtualserver_resource/vs-securework_arcadia.yaml

comments:
- verify VIP IPs in 'vs-securework_tea_coffee.yaml' and 'vs-securework_arcadia.yaml'
- optional deploy arcadia application