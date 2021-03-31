    #!/bin/bash
    
    kubectl delete -f k8s.nginx.org_virtualservers.yaml
    kubectl delete -f k8s.nginx.org_virtualserverroutes.yaml
    kubectl delete -f k8s.nginx.org_policies.yaml
    
    kubectl delete -f appprotect.f5.com_aplogconfs.yaml
    kubectl delete -f appprotect.f5.com_appolicies.yaml
    kubectl delete -f appprotect.f5.com_apusersigs.yaml