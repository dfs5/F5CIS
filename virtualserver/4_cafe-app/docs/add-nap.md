
  kubectl delete -f 4_test-example/cafe-ingress.yaml
  kubectl create -f 4_test-example/cafe-ingress-waf.yaml

comments:
- in 'cafe-ingress-waf.yaml' annotetions for WAF policy added:
  annotations:
    appprotect.f5.com/app-protect-policy: "default/dataguard-blocking"
    appprotect.f5.com/app-protect-enable: "True"