# ロードバランサ

ロードバランサとして MetalLB を構築する。

## ファイアウォール

すべてのノードで ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=7946/{tcp,udp}
firewall-cmd --permanent --zone=internal --add-port=7946/{tcp,udp}
firewall-cmd --reload
```

## インストール

MetalLB を構築する。

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```

```
namespace/metallb-system created
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io created
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io created
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/servicel2statuses.metallb.io created
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
configmap/metallb-excludel2 created
secret/metallb-webhook-cert created
service/metallb-webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration created
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods -n metallb-system
```

```
NAME                          READY   STATUS    RESTARTS   AGE
controller-8694df9d9b-4j98j   1/1     Running   0          93s
speaker-fxd5t                 1/1     Running   0          93s
speaker-l9hrl                 1/1     Running   0          93s
speaker-lg7h6                 1/1     Running   0          93s
```

## 環境確認

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n metallb-system -o wide
```

```
NAME                              READY   STATUS    RESTARTS   AGE    IP               NODE                    NOMINATED NODE   READINESS GATES
pod/controller-8694df9d9b-4j98j   1/1     Running   0          8m7s   172.17.255.141   worker01.home.local     <none>           <none>
pod/speaker-fxd5t                 1/1     Running   0          8m7s   172.16.0.32      worker02.home.local     <none>           <none>
pod/speaker-l9hrl                 1/1     Running   0          8m7s   172.16.0.31      worker01.home.local     <none>           <none>
pod/speaker-lg7h6                 1/1     Running   0          8m7s   172.16.0.11      controller.home.local   <none>           <none>

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/metallb-webhook-service   ClusterIP   10.102.236.145   <none>        443/TCP   8m7s   component=controller

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE    CONTAINERS   IMAGES                            SELECTOR
daemonset.apps/speaker   3         3         3       3            3           kubernetes.io/os=linux   8m7s   speaker      quay.io/metallb/speaker:v0.14.8   app=metallb,component=speaker

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES                               SELECTOR
deployment.apps/controller   1/1     1            1           8m7s   controller   quay.io/metallb/controller:v0.14.8   app=metallb,component=controller

NAME                                    DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES                               SELECTOR
replicaset.apps/controller-8694df9d9b   1         1         1       8m7s   controller   quay.io/metallb/controller:v0.14.8   app=metallb,component=controller,pod-template-hash=8694df9d9b
```

ロードバランサリソースを確認する。

```sh
kubectl get service -n ingress-nginx
```

```
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.110.103.111   <pending>     80:32157/TCP,443:31755/TCP   22h
ingress-nginx-controller-admission   ClusterIP      10.107.109.72    <none>        443/TCP                      22h
```

### コンテナ

コンテナを確認する。

```sh
crictl ps --name speaker
```

```
CONTAINER           IMAGE                                                                                             CREATED             STATE               NAME                ATTEMPT             POD ID              POD
ab264a952ca79       quay.io/metallb/speaker@sha256:c50578d7aaf9147c95e828d6d4a38d817de554cd6f8dd5512790eae48d18129e   12 minutes ago      Running             speaker             0                   64128683419d1       speaker-lg7h6
```

コンテナイメージを確認する。

```sh
crictl images | grep metal
```

```
quay.io/metallb/speaker                              v0.14.8             50d3d2d1712d7       120MB
```

## IP プール

ロードバランサに割り当てる IP アドレスを追加する。

```sh
cat >metallb_ip_pool.yaml <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: public-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.16.0.100-172.16.0.149
EOF

kubectl apply -f metallb_ip_pool.yaml
```

```
ipaddresspool.metallb.io/public-pool created
```

IP アドレスを L2 モードでアドバタイズする。

```sh
cat >metallb_ip_adv.yaml <<EOF
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: public
  namespace: metallb-system
spec:
  ipAddressPools:
  - public-pool
EOF

kubectl apply -f metallb_ip_adv.yaml
```

```
l2advertisement.metallb.io/public created
```

## 環境確認

### Kubernetes リソース

ロードバランサリソースを確認する。

```sh
kubectl get service -n ingress-nginx
```

```
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.110.103.111   172.16.0.100   80:32157/TCP,443:31755/TCP   23h
ingress-nginx-controller-admission   ClusterIP      10.107.109.72    <none>         443/TCP                      23h
```

```sh
kubectl describe service ingress-nginx-controller -n ingress-nginx
```

```
Name:                     ingress-nginx-controller
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=ingress-nginx
                          app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
                          app.kubernetes.io/version=1.11.2
Annotations:              metallb.universe.tf/ip-allocated-from-pool: public-pool
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.110.103.111
IPs:                      10.110.103.111
LoadBalancer Ingress:     172.16.0.100 (VIP)
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  32157/TCP
Endpoints:                172.17.255.140:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  31755/TCP
Endpoints:                172.17.255.140:443
Session Affinity:         None
External Traffic Policy:  Local
Internal Traffic Policy:  Cluster
HealthCheck NodePort:     31368
Events:
  Type    Reason        Age    From                Message
  ----    ------        ----   ----                -------
  Normal  IPAllocated   3m26s  metallb-controller  Assigned IP ["172.16.0.100"]
  Normal  nodeAssigned  2m42s  metallb-speaker     announcing from node "worker01.home.local" with protocol "layer2"
```

## 動作確認

ポッドを作成する。

```sh
kubectl create deployment demo-lb --image=httpd --port=80
```

```
deployment.apps/demo-lb created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```
NAME                       READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
demo-6c87cdc7bf-57f84      1/1     Running   3          22h   172.17.51.139    worker02.home.local   <none>           <none>
demo-lb-858d8b6884-fq656   1/1     Running   0          79s   172.17.255.142   worker01.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo-lb --type=LoadBalancer
```

```
service/demo-lb exposed
```

サービスを確認する。

```sh
kubectl get service demo-lb
```

```
NAME      TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
demo-lb   LoadBalancer   10.107.70.254   172.16.0.101   80:31663/TCP   34s
```

接続確認する。

```sh
curl http://172.16.0.101
```

```
<html><body><h1>It works!</h1></body></html>
```
