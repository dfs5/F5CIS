Remove Proxy Mode configuration and deploy tcp TransportServer.

    kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/master/deployments/common/nginx-config.yaml
    kubectl apply -f https://raw.githubusercontent.com/dfs5/F5CIS/master/CIS/3_CIS/ts_tcp.yaml

Access cafe-app from browser:

    https://cafe.example.com:8443/coffee

Verify NAP is running:

    https://cafe.example.com:8443/coffee?dfs=<script>

Note: Delete custom resource when you are finished.

    kubectl delete transportserver transport-server -n nginx-ingress