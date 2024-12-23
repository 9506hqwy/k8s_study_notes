# Gateway

## インストール

Ingress Gateway Fabric を構築する。

Gateway API のカスタムリソース定義を作成する。

```sh
kubectl kustomize "https://github.com/nginxinc/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | \
    kubectl apply -f -
```

```text
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
```

Nginx Gateway Fabric のカスタムリソース定義を作成する。

```sh
kubectl apply -f "https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.5.1/deploy/crds.yaml"
```

```text
customresourcedefinition.apiextensions.k8s.io/clientsettingspolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxgateways.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxproxies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/observabilitypolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/snippetsfilters.gateway.nginx.org created
```

Nginx Gateway Fabric をデプロイする。

```sh
kubectl apply -f "https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.5.1/deploy/default/deploy.yaml"
```

```text
namespace/nginx-gateway created
serviceaccount/nginx-gateway created
clusterrole.rbac.authorization.k8s.io/nginx-gateway created
clusterrolebinding.rbac.authorization.k8s.io/nginx-gateway created
configmap/nginx-includes-bootstrap created
service/nginx-gateway created
deployment.apps/nginx-gateway created
gatewayclass.gateway.networking.k8s.io/nginx created
nginxgateway.gateway.nginx.org/nginx-gateway-config created
```

デプロイを確認する。

```sh
kubectl -n nginx-gateway get pods -o wide
```

```text
NAME                                 READY   STATUS    RESTARTS   AGE    IP               NODE                  NOMINATED NODE   READINESS GATES
pod/nginx-gateway-594dc7977b-m2l7c   2/2     Running   0          107s   172.17.255.180   worker01.home.local   <none>           <none>

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE    SELECTOR
service/nginx-gateway   LoadBalancer   10.105.75.170   172.16.0.103   80:32483/TCP,443:30953/TCP   107s   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS            IMAGES                                                                                          SELECTOR
deployment.apps/nginx-gateway   1/1     1            1           107s   nginx-gateway,nginx   ghcr.io/nginxinc/nginx-gateway-fabric:1.5.1,ghcr.io/nginxinc/nginx-gateway-fabric/nginx:1.5.1   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                                       DESIRED   CURRENT   READY   AGE    CONTAINERS            IMAGES                                                                                          SELECTOR
replicaset.apps/nginx-gateway-594dc7977b   1         1         1       107s   nginx-gateway,nginx   ghcr.io/nginxinc/nginx-gateway-fabric:1.5.1,ghcr.io/nginxinc/nginx-gateway-fabric/nginx:1.5.1   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway,pod-template-hash=594dc7977b
```

## 動作確認

ポッドを作成する。

```sh
kubectl create deployment demo --image=httpd --port=80
```

```text
deployment.apps/demo created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```text
NAME                    READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
demo-6c87cdc7bf-kbhp4   1/1     Running   0          9s    172.17.51.135   worker02.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo
```

```text
kubectl expose deployment demo
service/demo exposed
```

サービスを確認する。

```sh
kubectl get service
```

```text
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
demo         ClusterIP   10.101.87.6   <none>        80/TCP    7s
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   86d
```

Gateway を作成する。

```sh
cat >gateway.yaml <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: demo
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
spec:
  parentRefs:
  - name: demo
  hostnames:
  - demo.localdev.me
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: demo
      port: 80
EOF

kubectl apply -f gateway.yaml
```

```text
gateway.gateway.networking.k8s.io/demo created
httproute.gateway.networking.k8s.io/demo-route created
```

Gateway を確認する。

```sh
kubectl get gateway.gateway.networking.k8s.io
```

```text
NAME   CLASS   ADDRESS        PROGRAMMED   AGE
demo   nginx   172.16.0.103   True         25s
```

ルーティングを確認する。

```sh
kubectl get httproute.gateway.networking.k8s.io/demo-route
```

```text
NAME         HOSTNAMES              AGE
demo-route   ["demo.localdev.me"]   100s
```

接続確認する。

```sh
curl --resolve demo.localdev.me:80:172.16.0.103 http://demo.localdev.me/
```

```text
<html><body><h1>It works!</h1></body></html>
```
