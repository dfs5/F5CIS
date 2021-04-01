
    kubectl delete -f 1_cis_controller/f5-cis-deployment.yaml 
    kubectl create -f 1_cis_controller/f5-cis-deployment_vs.yaml

comments: 
- "--ingress-link-mode=true"    removed in 'f5-cis-deployment_vs.yaml'