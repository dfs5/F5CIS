#!/bin/bash

kubectl apply -f k8s.nginx.org_virtualservers.yaml
kubectl apply -f k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f k8s.nginx.org_policies.yaml

kubectl apply -f appprotect.f5.com_aplogconfs.yaml
kubectl apply -f appprotect.f5.com_appolicies.yaml
kubectl apply -f appprotect.f5.com_apusersigs.yaml