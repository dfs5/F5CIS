
    kubectl delete -f 1_cis_controller/f5-cis-deployment_vs.yaml
    kubectl create -f 1_cis_controller/f5-cis-deployment.yaml


comments: 
- "--ingress-link-mode=true"    added in 'f5-cis-deployment.yaml'