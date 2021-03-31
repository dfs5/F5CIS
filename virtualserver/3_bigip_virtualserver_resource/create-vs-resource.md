#!/bin/bash

kubectl create -f terminate-tls.yaml
kubectl create -f vs-securework_tea_coffee.yaml

#optional:
#kubectl create -f vs-securework_arcadia.yaml

comments:
- verify VIP IPs in 'vs-securework_tea_coffee.yaml' and 'vs-securework_arcadia.yaml' are different!
- optional deploy arcadia application