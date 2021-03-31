#!/bin/bash

kubectl delete -f terminate-tls.yaml
kubectl delete -f vs-securework_tea_coffee.yaml

#optional:
#kubectl delete -f vs-securework_arcadia.yaml