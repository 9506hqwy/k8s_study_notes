# Gateway (Calico)

## インストール

Gateway API を有効化する。

```sh
kubectl apply -f - <<EOF
apiVersion: operator.tigera.io/v1
kind: GatewayAPI
metadata:
  name: default
EOF
```

```text
gatewayapi.operator.tigera.io/default created
```

Gateway API が有効になったことを確認する。

```sh
kubectl get tigerastatus gatewayapi
```

```text
NAME         AVAILABLE   PROGRESSING   DEGRADED   SINCE
gatewayapi   True        False         False      16m
```

リソースの種別を確認する。

```sh
kubectl api-resources | grep gateway.networking.k8s.io
```

```text
backendlbpolicies                    blbpolicy                                                 gateway.networking.k8s.io/v1alpha2          true         BackendLBPolicy
backendtlspolicies                   btlspolicy                                                gateway.networking.k8s.io/v1alpha3          true         BackendTLSPolicy
gatewayclasses                       gc                                                        gateway.networking.k8s.io/v1                false        GatewayClass
gateways                             gtw                                                       gateway.networking.k8s.io/v1                true         Gateway
grpcroutes                                                                                     gateway.networking.k8s.io/v1                true         GRPCRoute
httproutes                                                                                     gateway.networking.k8s.io/v1                true         HTTPRoute
referencegrants                      refgrant                                                  gateway.networking.k8s.io/v1beta1           true         ReferenceGrant
tcproutes                                                                                      gateway.networking.k8s.io/v1alpha2          true         TCPRoute
tlsroutes                                                                                      gateway.networking.k8s.io/v1alpha2          true         TLSRoute
udproutes                                                                                      gateway.networking.k8s.io/v1alpha2          true         UDPRoute
```

GatewayClass があることを確認する。

```sh
kubectl get gatewayclass -o=jsonpath='{.items[*].spec}' | jq
```

```json
{
  "controllerName": "gateway.nginx.org/nginx-gateway-controller",
  "parametersRef": {
    "group": "gateway.nginx.org",
    "kind": "NginxProxy",
    "name": "nginx-gateway-proxy-config",
    "namespace": "nginx-gateway"
  }
}
{
  "controllerName": "gateway.envoyproxy.io/gatewayclass-controller",
  "parametersRef": {
    "group": "gateway.envoyproxy.io",
    "kind": "EnvoyProxy",
    "name": "envoy-proxy-config",
    "namespace": "tigera-gateway"
  }
}
```

## 動作確認

[ロードバランサ](./loadbalancer.md) を構築したあとでポッドを作成する。

```sh
kubectl create deployment demo-gateway --image=httpd --port=80
```

```text
deployment.apps/demo-gateway created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```text
NAME                            READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
demo-gateway-848c7cbd64-d7vfv   1/1     Running   0          21s   172.17.51.146   worker02.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo-gateway
```

```text
service/demo-gateway exposed
```

サービスを確認する。

```sh
kubectl get service
```

```text
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
demo-gateway   ClusterIP   10.108.121.190   <none>        80/TCP    7s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP   43d
```

Gateway を作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: demo
spec:
  gatewayClassName: tigera-gateway-class
  listeners:
  - name: http
    port: 80
    protocol: HTTP
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-gateway
spec:
  parentRefs:
  - name: demo
  hostnames:
  - demo-gateway.localdev.me
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: demo-gateway
      port: 80
EOF
```

```text
gateway.gateway.networking.k8s.io/demo created
httproute.gateway.networking.k8s.io/demo-gateway created
```

Gateway を確認する。

```sh
kubectl get gateway.gateway.networking.k8s.io
```

```text
NAME   CLASS                  ADDRESS        PROGRAMMED   AGE
demo   tigera-gateway-class   172.16.0.100   True         14s
```

ルーティングを確認する。

```sh
kubectl get httproute.gateway.networking.k8s.io
```

```text
NAME           HOSTNAMES                      AGE
demo-gateway   ["demo-gateway.localdev.me"]   31s
```

接続確認する。

```sh
curl --resolve demo-gateway.localdev.me:80:172.16.0.100 http://demo-gateway.localdev.me/
```

```text
<html><body><h1>It works!</h1></body></html>
```
