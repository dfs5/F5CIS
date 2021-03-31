#!/bin/bash

kubectl describe -f cafe-ns.yaml
kubectl describe -f cafe-secret.yaml
kubectl describe -f coffee.yaml
kubectl describe -f cafe-ingress.yaml
