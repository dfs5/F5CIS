    #!/bin/bash
    
    kubectl create -f k8s.nginx.org_virtualservers.yaml
    kubectl create -f k8s.nginx.org_virtualserverroutes.yaml
    kubectl create -f k8s.nginx.org_policies.yaml
    
    kubectl create -f appprotect.f5.com_aplogconfs.yaml
    kubectl create -f appprotect.f5.com_appolicies.yaml
    kubectl create -f appprotect.f5.com_apusersigs.yaml