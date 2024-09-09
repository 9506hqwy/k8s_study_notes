# Ingress Controller

TODO: テスト

## インストール

Ingress NGINX Controller を構築する。

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

```
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

すべての Pod が起動するまで待つ。

```sh
kubectl get pods --namespace=ingress-nginx
```

```
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-lvph9        0/1     Completed   0          5m48s
ingress-nginx-admission-patch-m5gnp         0/1     Completed   0          5m47s
ingress-nginx-controller-6d675964ff-sw5vq   1/1     Running     0          5m48s
```

## ファイアウォール

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=8443/tcp
firewall-cmd --permanent --zone=internal --add-port=8443/tcp
firewall-cmd --reload
```

ワーカノードのファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

## 環境の確認

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n ingress-nginx
```

```
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-lvph9        0/1     Completed   0          9m43s
pod/ingress-nginx-admission-patch-m5gnp         0/1     Completed   0          9m42s
pod/ingress-nginx-controller-6d675964ff-sw5vq   1/1     Running     0          9m43s

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.110.35.192   <pending>     80:32108/TCP,443:31373/TCP   9m45s
service/ingress-nginx-controller-admission   ClusterIP      10.102.239.74   <none>        443/TCP                      9m44s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           9m43s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-6d675964ff   1         1         1       9m43s

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           35s        9m43s
job.batch/ingress-nginx-admission-patch    Complete   1/1           36s        9m43s
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
                          app.kubernetes.io/version=1.10.1
Annotations:              <none>
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.110.35.192
IPs:                      10.110.35.192
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  32108/TCP
Endpoints:                192.168.255.133:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  31373/TCP
Endpoints:                192.168.255.133:443
Session Affinity:         None
External Traffic Policy:  Local
HealthCheck NodePort:     31299
Events:                   <none>
```

```sh
kubectl describe service ingress-nginx-controller-admission -n ingress-nginx
```

```
Name:              ingress-nginx-controller-admission
Namespace:         ingress-nginx
Labels:            app.kubernetes.io/component=controller
                   app.kubernetes.io/instance=ingress-nginx
                   app.kubernetes.io/name=ingress-nginx
                   app.kubernetes.io/part-of=ingress-nginx
                   app.kubernetes.io/version=1.10.1
Annotations:       <none>
Selector:          app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.102.239.74
IPs:               10.102.239.74
Port:              https-webhook  443/TCP
TargetPort:        webhook/TCP
Endpoints:         192.168.255.133:8443
Session Affinity:  None
Events:            <none>
```

### コンテナ

ワーカノードで確認する。

```sh
crictl ps
```

```
CONTAINER           IMAGE                                                                                                               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
4dfd2eac981fd       registry.k8s.io/ingress-nginx/controller@sha256:959e313aceec9f38e18a329ca3756402959e84e63ae8ba7ac1ee48aec28d51b9    9 minutes ago       Running             controller                  0                   21a9c7c22613b       ingress-nginx-controller-6d675964ff-sw5vq
aa29acdbe4f62       gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af   2 hours ago         Running             kubernetes-bootcamp         0                   1a48ae5a4fe81       kubernetes-bootcamp-644c5687f4-4dc2n
0827f6e1271a9       0f80feca743f4a84ddda4057266092db9134f9af9e20e12ea6fcfe51d7e3a020                                                    2 hours ago         Running             csi-node-driver-registrar   1                   535ab6951f843       csi-node-driver-hslhz
08cea41e6cdfa       1a094aeaf1521e225668c83cbf63c0ec63afbdb8c4dd7c3d2aab0ec917d103de                                                    2 hours ago         Running             calico-csi                  1                   535ab6951f843       csi-node-driver-hslhz
3f9f709639180       4e42b6f329bc1d197d97f6d2a1289b9e9f4a9560db3a36c8cffb5e95e64e4b49                                                    2 hours ago         Running             calico-node                 2                   c1580caa7e31c       calico-node-dvv2b
943cad010f914       747097150317f99937cabea484cff90097a2dbd79e7eb348b71dc0af879883cd                                                    2 hours ago         Running             kube-proxy                  1                   63edbb657a53c       kube-proxy-kfqdc
```
