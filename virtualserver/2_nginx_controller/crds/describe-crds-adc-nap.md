
    kubectl describe -f crds/k8s.nginx.org_virtualservers.yaml
    kubectl describe -f crds/k8s.nginx.org_virtualserverroutes.yaml
    kubectl describe -f crds/k8s.nginx.org_policies.yaml
    
    kubectl describe -f appprotect.f5.com_aplogconfs.yaml
    kubectl describe -f appprotect.f5.com_appolicies.yaml
    kubectl describe -f appprotect.f5.com_apusersigs.yaml