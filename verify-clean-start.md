
# All output has to be empty.

kubectl get pod -A |grep nginx
kube-system            nginx-proxy-node3                            1/1     Running   13         22h

kubectl get pod -A |grep bigip

kubectl get ns |grep cafe

kubectl get ns |grep arcadia

kubectl get ns |grep nginx-ingress

kubectl get pod -n cafe

kubectl get pod -n arcadia

kubectl get ingress -A

kubectl get svc -A |grep cafe-svc

kubectl get svc -A |grep tea-svc

kubectl get svc -A |grep nginx-ingress-ingresslink

kubectl get svc -n arcadia

kubectl get secret -n kube-system |grep bigip-login