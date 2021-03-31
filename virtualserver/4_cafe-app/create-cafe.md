    #!/bin/bash
    
    kubectl create -f cafe-ns.yaml
    kubectl create -f cafe-secret.yaml
    kubectl create -f coffee.yaml
    kubectl create -f cafe-ingress.yaml