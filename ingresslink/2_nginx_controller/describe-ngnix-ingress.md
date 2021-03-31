    
    #describe kubernetes bigip container connecter, authentication and RBAC     resources

    kubectl describe -f ns-and-sa.yaml
    kubectl describe -f rbac/rbac.yaml
    kubectl describe -f rbac/ap-rbac.yaml
    kubectl describe -f default-server-secret.yaml
    kubectl describe -f nginx-config.yaml
    kubectl describe -f ingress-class.yaml
    kubectl describe -f nginx-ingress.yaml
    kubectl describe -f nginx-service.yaml