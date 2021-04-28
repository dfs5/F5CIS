# How to setup private registry for a Kubernetes cluster.

## Preparation

Create on a CA a cert/key pair for https access to registry.domain.name and export it with the CA cert.

On each worker node:

    vi /etc/hosts
    10.24.1.10 registry.dfslab.local

Import CA cert into --> /etc/ssl/certs

    sudo systemctl daemon-reload
    sudo systemctl reload docker

 
I gonna run my registry on the master node. So putting the ssl certificates there.

    ./cert/registry.dfslab.local.crt
    . /cert/registry.dfslab.local.key

## Create a docker hosting the registry.

	vi docker-compose.yaml

    registry:
     restart: always
     image: registry:2
     ports:
       - "5000:5000"
     environment:
       REGISTRY_HTTP_TLS_CERTIFICATE: /cert/registry.dfslab.local.crt
       REGISTRY_HTTP_TLS_KEY: /cert/registry.dfslab.local.key
       REGISTRY_AUTH: htpasswd
       REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
       REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
     volumes:
       - ./data:/var/lib/registry
       - ./cert:/cert
       - ./auth:/auth

	docker-compose up -d

 
## On each worker node verify you can login to your repo:

    sudo docker login registry.dfslab.local:5000
 
Verify config.json was created:

    sudo cat ~/.docker/config.json

## In order to access the image from a Kubernetes deployment a secret is needed. On K8S master generate base64 for kubernetes Secret:

    fod@master:~$ cat ~/.docker/config.json | base64
    ewoJIm[...]]y42IChsaW51eCkiCgl9Cn0=

    vi registry-secret.yaml
    apiVersion: v1
    kind: Secret
    metadata:
     name: regcred
     namespace: nginx-ingress
    data:
     .dockerconfigjson: eewoJIm[...]]y42IChsaW51eCkiCgl9Cn0=
    type: kubernetes.io/dockerconfigjson

	kubectl create -f registry-secret.yaml

## If you need to build a new NGINX image execute the make command as described here:
https://docs.nginx.com/nginx-ingress-controller/app-protect/installation/#build-the-docker-image

    # example for NGINX IC for Kubernetes:
    make debian-image-nap-plus PREFIX=registry.dfslab.local:5000/nginx-plus-ingress TARGET=container

## If you want to upload an existing image in your private repository

    docker tag registry.dfslab.local:5000/nginx-plus-ingress:edge
    docker push registry.dfslab.local:5000/nginx-plus-ingress:edge

## Point to the private registry in your deployments and don’t forget to insert secret in the END of your deployment file.

    Example:
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: nginx-ingress
      namespace: nginx-ingress
    spec:
      selector:
        matchLabels:
          app: nginx-ingress
      template:
        metadata:
          labels:
            app: nginx-ingress
         #annotations:
           #prometheus.io/scrape: "true"
           #prometheus.io/port: "9113"
        spec:
          serviceAccountName: nginx-ingress
          containers:
            - image: registry.dfslab.local:5000/nginx-plus-ingress:edge
              imagePullPolicy: Always
              name: nginx-plus-ingress
              ports:
              - name: http
                containerPort: 80
                hostPort: 80
              - name: https
                containerPort: 443
                hostPort: 443
              - name: readiness-port
                containerPort: 8081
            #- name: prometheus
              #containerPort: 9113
              readinessProbe:
                httpGet:
                  path: /nginx-ready
                  port: readiness-port
                periodSeconds: 1
              securityContext:
                allowPrivilegeEscalation: true
                runAsUser: 101 #nginx
                capabilities:
                  drop:
                  - ALL
                  add:
                  - NET_BIND_SERVICE
              env:
              - name: POD_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              args:
                - -nginx-plus
                - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
                - -default-server-tls-secret=$(POD_NAMESPACE)/  default-server-secret
                - -enable-app-protect
              #  - -watch-namespace=cafe, arcadia
              #  - -watch-namespace=arcadia        
              #- -v=3 # Enables extensive logging. Useful for   troubleshooting.
              #- -report-ingress-status
              #- -external-service=nginx-ingress
              #- -enable-prometheus-metrics
              #- -global-configuration=$(POD_NAMESPACE)/nginx-configuration
          imagePullSecrets:
            - name: regcred
 

Sources:

https://theithollow.com/2020/03/03/use-a-private-registry-with-kubernetes/

https://blog.cloudhelix.io/using-a-private-docker-registry-with-kubernetes-f8d5f6b8f646

