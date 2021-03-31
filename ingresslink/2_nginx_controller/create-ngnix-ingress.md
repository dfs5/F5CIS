#!/bin/bash

#create kubernetes bigip container connecter, authentication and RBAC
kubectl apply -f crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f crds/k8s.nginx.org_policies.yaml
kubectl apply -f crds/appprotect.f5.com_aplogconfs.yaml
kubectl apply -f crds/appprotect.f5.com_appolicies.yaml
kubectl apply -f crds/appprotect.f5.com_apusersigs.yaml

kubectl create -f ns-and-sa.yaml
kubectl create -f rbac/rbac.yaml
kubectl create -f rbac/ap-rbac.yaml
kubectl create -f default-server-secret.yaml
kubectl create -f nginx-config.yaml
kubectl create -f ingress-class.yaml
kubectl create -f nginx-ingress.yaml
kubectl create -f nginx-service.yaml