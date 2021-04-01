    #!/bin/bash

    #delete kubernetes nginx-ingress container, authentication and RBAC
    kubectl delete -f ns-and-sa.yaml
    kubectl delete -f rbac/rbac.yaml
    kubectl delete -f rbac/ap-rbac.yaml
    kubectl delete -f ingress-class.yaml