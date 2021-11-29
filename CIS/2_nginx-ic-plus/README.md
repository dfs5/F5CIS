## Prerequisites 
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#prerequisites)

We will pull deployment files directly from the official nginxinc/kubernetes-ingress repo.
## 1. Configure RBAC and Namespace
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#1-configure-rbac)

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/ns-and-sa.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/rbac/rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/rbac/ap-rbac.yaml 
## 2. Create Common Resources
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#2-create-common-resources)

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/default-server-secret.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/ingress-class.yaml

## Create Custom Resources
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#create-custom-resources)

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/k8s.nginx.org_virtualservers.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/k8s.nginx.org_virtualserverroutes.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/k8s.nginx.org_transportservers.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/k8s.nginx.org_policies.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/k8s.nginx.org_globalconfigurations.yaml

## Resources for NGINX App Protect
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#resources-for-nginx-app-protect)

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/appprotect.f5.com_aplogconfs.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/appprotect.f5.com_appolicies.yaml
    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/crds/appprotect.f5.com_apusersigs.yaml

## 3. Deploy syslog service to monitor NAP request blocking. (This is optional)

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/nap-log.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/syslog-service.yaml

## 4. Deploy the NGINX+ IC
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#3-deploy-the-ingress-controller)

Note: Update the nginx-plus-ingress.yaml with the container image that you have built. Howto guide can be found here: https://github.com/dfs5/F5CIS/tree/master/PrivateRegistry
##### e.g. - image: registry.dfslab.local:5000/nginx-plus-ingress:v1.11.1
Note: Don't forget to create secret for your private registry (my-registry-secret.yaml): 

    apiVersion: v1
    kind: Secret
    metadata:
     name: my-registry-secret
     namespace: nginx-ingress
    data:
     .dockerconfigjson: base64encoded
    type: kubernetes.io/dockerconfigjson

...and add that secret in the end of the deployment declaration.
##### imagePullSecrets:
#####   - name: my-registry-secret
Note: Enable N+ with App Protect and allow access to N+ dashboard from 10.24.0.0/16 network. Set to 0.0.0.0/0 for any.
##### args:
#####       - -nginx-plus
#####       - -enable-app-protect
#####       - -nginx-status-allow-cidrs=10.24.0.0/16  <-- adjust to match your environment
#####       - -ingresslink=nginx-ingress    <-- for IngressLink
#####       - -report-ingress-status        <-- for IngressLink
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/nginx-plus-ingress-health.yaml

## 4.2 Check that the Ingress Controller is Running
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#32-check-that-the-ingress-controller-is-running)

##### $ kubectl get pods --namespace=nginx-ingress
##### NAME                             READY   STATUS    RESTARTS   AGE
##### nginx-ingress-84b97c486c-csrqp   1/1     Running   0          2m26s

## 5. Get Access to the Ingress Controller
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#4-get-access-to-the-ingress-controller)

Note: Using NodePort mode

    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/2_nginx-ic-plus/nodeport_dashboard.yaml
    kubectl get svc -n nginx-ingress

Verify NGINX IC is running and listening on http/https

    curl http://nodeIP:30001
    curl https://nodeIO:30002   <-- default certificate used
        <html>
        <head><title>404 Not Found</title></head>
        <body>
        <center><h1>404 Not Found</h1></center>
        <hr><center>nginx/1.19.5</center>
        </body>
        </html>
        
Verify NGINX IC dashboard is running (configuration is still empty) 

    In browser: http://nodeIP:30003/dashboard.html#
    
### Go back to [1_cafe-app](https://github.com/dfs5/F5CIS/tree/master/CIS/1_cafe-app#deploy-nginx-ic-first-befor-proceeding-2_nginx-ic-plus) and deploy Ingress service befor you proceed on '3_CIS' !!!

## Appendix: Uninstall the Ingress Controller
(https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/#uninstall-the-ingress-controller)

    kubectl delete namespace nginx-ingress
    kubectl delete clusterrole nginx-ingress
    kubectl delete clusterrolebinding nginx-ingress
    kubectl delete clusterrole nginx-ingress-app-protect
    kubectl delete clusterrolebinding nginx-ingress-app-protect
    kubectl delete IngressClass nginx
    kubectl delete CustomResourceDefinition virtualservers.k8s.nginx.org
    kubectl delete CustomResourceDefinition virtualserverroutes.k8s.nginx.org
    kubectl delete CustomResourceDefinition transportservers.k8s.nginx.org
    kubectl delete CustomResourceDefinition policies.k8s.nginx.org
    kubectl delete CustomResourceDefinition globalconfigurations.k8s.nginx.org
    kubectl delete CustomResourceDefinition aplogconfs.appprotect.f5.com
    kubectl delete CustomResourceDefinition appolicies.appprotect.f5.com
    kubectl delete CustomResourceDefinition apusersigs.appprotect.f5.com

