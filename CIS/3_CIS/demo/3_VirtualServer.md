In this lab we will disable tls termination in NGINX IC and move it to BIG-IP. On the BIG-IP we will have a standard HTTP VIP frontending our K8S cluster. NGINX IC is still doing NAP.

    kubectl delete Ingress cafe-ingress -n cafe
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/cafe-ingress-waf_noTLS.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vsp_nginx-cafe-terminate-tls.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/vs_nginx-cafe.yaml

See CR configuration in BIG-IP UI!!!

Access cafe-app from browser:

    http://cafe.example.com/coffee

Verify NAP is still running:

    https://cafe.example.com/coffee?dfs=<script>

Note: Delete custom resource when you are finished and check configuration is removed from BIG-IP.

    kubectl delete -f vs_nginx-cafe.yaml
    kubectl delete -f vsp_nginx-cafe-terminate-tls.yaml
