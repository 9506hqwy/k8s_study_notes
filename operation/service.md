# サービス

## ログレベルを変更

ログレベルを変更できるロールを作成する。

```sh
kubectl create clusterrole \
    debug-node-proxy \
    --verb=update \
    --resource=nodes/proxy
```

```text
clusterrole.rbac.authorization.k8s.io/debug-node-proxy created
```

サービスアカウントにログレベルを変更する権限を付与する。

```sh
kubectl create clusterrolebinding \
    debug-node-proxy \
    --clusterrole=debug-node-proxy \
    --serviceaccount=default:default
```

```text
clusterrolebinding.rbac.authorization.k8s.io/debug-node-proxy created
```

トークンを作成する。

```sh
kubectl create token default
```

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IlM...
```

kubelet のログレベルを変更する。

```sh
curl -k -H "Authorization: Bearer $TOKEN" -d 5 -X PUT  https://controller:10250/debug/flags/v
curl -k -H "Authorization: Bearer $TOKEN" -d 5 -X PUT  https://worker01:10250/debug/flags/v
curl -k -H "Authorization: Bearer $TOKEN" -d 5 -X PUT  https://worker02:10250/debug/flags/v
```

```text
successfully set klog.logging.verbosity to 5
```

kube-proxy のログレベルを変更する。

```sh
kubectl -n kube-system get configmap kube-proxy -o yaml | \
    sed -e 's/enableProfiling: false/enableProfiling: true/'  | \
    kubectl apply -f -
```

```text
Warning: resource configmaps/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/kube-proxy configured
```

ロールアウトする。

```sh
kubectl -n kube-system rollout restart daemonset/kube-proxy
```

```text
daemonset.apps/kube-proxy restarted
```

```sh
curl -d 5 -X PUT  http://127.0.0.1:10249/debug/flags/v
ssh worker01 curl -d 5 -X PUT  http://127.0.0.1:10249/debug/flags/v
ssh worker02 curl -d 5 -X PUT  http://127.0.0.1:10249/debug/flags/v
```

## ClusterIP

ポッドを作成する。

```sh
kubectl create deployment demo-svc --image=nginx --port=80
```

```text
deployment.apps/demo-svc created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```text
NAME                          READY   STATUS    RESTARTS   AGE     IP               NODE                  NOMINATED NODE   READINESS GATES
demo-svc-9bf48d8-nhkjq        1/1     Running   0          6m54s   172.17.51.131    worker02.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo-svc
```

```text
service/demo-svc exposed
```

サービスを確認する。

```sh
kubectl get service -o wide
```

```text
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE   SELECTOR
demo-svc     ClusterIP      10.102.24.177   <none>         80/TCP         15s   app=demo-svc
```

nftables のルールセットは以下が設定される(`nft list ruleset ip`)。

```text
chain KUBE-SERVICES {
    meta l4proto tcp ip daddr 10.102.24.177  tcp dport 80 counter packets 0 bytes 0 jump KUBE-SVC-B4ZAZG5KKEIRBPYQ
}

chain KUBE-SVC-B4ZAZG5KKEIRBPYQ {
    meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.102.24.177  tcp dport 80 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    counter packets 0 bytes 0 jump KUBE-SEP-JP5VNVBXFICNHQNY
}

chain KUBE-SEP-JP5VNVBXFICNHQNY {
    ip saddr 172.17.51.131  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.51.131:80
}
```

接続確認する。

```sh
kubectl port-forward svc/demo-svc 8080:80
```

```text
Forwarding from 127.0.0.1:8080 -> 80
```

```sh
curl http://127.0.0.1:8080/
```

HTML が返却される。

## NodePort

サービスを作成する。

```sh
kubectl expose deployment demo-svc --name=demo-svc-nodeport --type=NodePort
```

```text
service/demo-svc-nodeport exposed
```

サービスを確認する。

```sh
kubectl get service -o wide
```

```text
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE   SELECTOR
demo-svc-nodeport   NodePort       10.98.22.211    <none>         80:30221/TCP   24s   app=demo-svc
```

nftables のルールセットは以下が設定される(`nft list ruleset ip`)。

```text
chain KUBE-SERVICES {
    meta l4proto tcp ip daddr 10.98.22.211  tcp dport 80 counter packets 0 bytes 0 jump KUBE-SVC-RTTGOT4DPK46RZJC
}

chain KUBE-NODEPORTS {
    meta l4proto tcp  tcp dport 30221 counter packets 0 bytes 0 jump KUBE-EXT-RTTGOT4DPK46RZJC
}

chain KUBE-EXT-RTTGOT4DPK46RZJC {
    counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    counter packets 0 bytes 0 jump KUBE-SVC-RTTGOT4DPK46RZJC
}

chain KUBE-SVC-RTTGOT4DPK46RZJC {
    meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.98.22.211  tcp dport 80 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    counter packets 0 bytes 0 jump KUBE-SEP-YLLUZU2E5ORDBVRG
}

chain KUBE-SEP-YLLUZU2E5ORDBVRG {
    ip saddr 172.17.51.131  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.51.131:80
}
```

接続確認する。

```sh
curl http://controller:30221
curl http://worker01:30221
curl http://worker02:30221
```

HTML が返却される。

## LoadBalancer

サービスを作成する。

```sh
kubectl expose deployment demo-svc --name=demo-svc-lb --type=LoadBalancer
```

```text
service/demo-svc-lb exposed
```

サービスを確認する。

```sh
kubectl get service -o wide
```

```text
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE    SELECTOR
demo-svc-lb         LoadBalancer   10.100.155.216   172.16.0.103   80:30199/TCP   11s    app=demo-svc
```

nftables のルールセットは以下が設定される(`nft list ruleset ip`)。

```text
chain KUBE-SERVICES {
    meta l4proto tcp ip daddr 172.16.0.103  tcp dport 80 counter packets 0 bytes 0 jump KUBE-EXT-3Y4IBFDS3N4DHOKS
    meta l4proto tcp ip daddr 10.100.155.216  tcp dport 80 counter packets 0 bytes 0 jump KUBE-SVC-3Y4IBFDS3N4DHOKS
}

chain KUBE-NODEPORTS {
    meta l4proto tcp  tcp dport 30199 counter packets 0 bytes 0 jump KUBE-EXT-3Y4IBFDS3N4DHOKS
}

chain KUBE-EXT-3Y4IBFDS3N4DHOKS {
    counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    counter packets 0 bytes 0 jump KUBE-SVC-3Y4IBFDS3N4DHOKS
}

chain KUBE-SVC-3Y4IBFDS3N4DHOKS {
    meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.100.155.216  tcp dport 80 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    counter packets 0 bytes 0 jump KUBE-SEP-6IZIMKP56A2Q6QNT
}

chain KUBE-SEP-6IZIMKP56A2Q6QNT {
    ip saddr 172.17.51.131  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
    meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.51.131:80
}
```

接続確認する。

```sh
curl http://controller:30199
curl http://worker01:30199
curl http://worker02:30199
curl http://172.16.0.103
```

HTML が返却される。
