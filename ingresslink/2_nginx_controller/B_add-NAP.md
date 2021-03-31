
# the following requires a private repositotry in this case: registry.dfslab.local:5000
# link to guide

git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/
git checkout v1.10.1
make DOCKERFILE=appprotect/DockerfileWithAppProtectForPlus PREFIX=registry.dfslab.local:5000/nap-ingress

kubectl delete -f 2_nginx_controller/nginx-ingress-no-ingresslink.yaml

kubectl create -f ~/_registry/registry-secret.yaml

kubectl create -f 2_nginx_controller/nap/nginx-plus-ingress-nap.yaml
kubectl create -f 2_nginx_controller/nap/APPolicy.yaml

#comments:
#- first build a local registry where the nap image will go into
#- in 'nginx-plus-ingress-nap.yaml' added:
#imagePullSecrets:
#        - name: regcred
#- 'APPolicy' specs are build based on 'dataguard_blocking_blueprint.json'