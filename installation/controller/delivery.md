# デリバリー

CD ツールとして Argo CD を構築する。

## インストール

Argo CD 用の名前空間を作成する。

```sh
kubectl create namespace argocd
```

```text
namespace/argocd created
```

Argo CD を構築する。

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

```text
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods --namespace=argocd
```

```text
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          75s
argocd-applicationset-controller-578697b885-mjttj   1/1     Running   0          75s
argocd-dex-server-95477cdd-sxcq9                    1/1     Running   0          75s
argocd-notifications-controller-787447c77d-g8tgh    1/1     Running   0          75s
argocd-redis-5746c4c5fb-mp4gt                       1/1     Running   0          75s
argocd-repo-server-588c6f4648-45pj4                 1/1     Running   0          75s
argocd-server-656b9b6c6c-xv88c                      1/1     Running   0          75s
```

クライアントをインストールする。

```sh
mkdir -p $HOME/.local/bin
curl -L https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64 --output-dir $HOME/.local/bin -o argocd
chmod +x $HOME/.local/bin/argocd
```

## 環境確認

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n argocd -o wide
```

```text
NAME                                                    READY   STATUS    RESTARTS   AGE     IP            NODE                    NOMINATED NODE   READINESS GATES
pod/argocd-application-controller-0                     1/1     Running   0          6m23s   172.17.2.63   controller.home.local   <none>           <none>
pod/argocd-applicationset-controller-578697b885-mjttj   1/1     Running   0          6m23s   172.17.2.59   controller.home.local   <none>           <none>
pod/argocd-dex-server-95477cdd-sxcq9                    1/1     Running   0          6m23s   172.17.2.58   controller.home.local   <none>           <none>
pod/argocd-notifications-controller-787447c77d-g8tgh    1/1     Running   0          6m23s   172.17.2.60   controller.home.local   <none>           <none>
pod/argocd-redis-5746c4c5fb-mp4gt                       1/1     Running   0          6m23s   172.17.2.62   controller.home.local   <none>           <none>
pod/argocd-repo-server-588c6f4648-45pj4                 1/1     Running   0          6m23s   172.17.2.61   controller.home.local   <none>           <none>
pod/argocd-server-656b9b6c6c-xv88c                      1/1     Running   0          6m23s   172.17.2.57   controller.home.local   <none>           <none>

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE     SELECTOR
service/argocd-applicationset-controller          ClusterIP   10.97.208.101    <none>        7000/TCP,8080/TCP            6m24s   app.kubernetes.io/name=argocd-applicationset-controller
service/argocd-dex-server                         ClusterIP   10.99.195.163    <none>        5556/TCP,5557/TCP,5558/TCP   6m24s   app.kubernetes.io/name=argocd-dex-server
service/argocd-metrics                            ClusterIP   10.100.131.39    <none>        8082/TCP                     6m24s   app.kubernetes.io/name=argocd-application-controller
service/argocd-notifications-controller-metrics   ClusterIP   10.105.36.207    <none>        9001/TCP                     6m24s   app.kubernetes.io/name=argocd-notifications-controller
service/argocd-redis                              ClusterIP   10.97.13.6       <none>        6379/TCP                     6m24s   app.kubernetes.io/name=argocd-redis
service/argocd-repo-server                        ClusterIP   10.104.232.175   <none>        8081/TCP,8084/TCP            6m24s   app.kubernetes.io/name=argocd-repo-server
service/argocd-server                             ClusterIP   10.100.127.147   <none>        80/TCP,443/TCP               6m24s   app.kubernetes.io/name=argocd-server
service/argocd-server-metrics                     ClusterIP   10.107.241.223   <none>        8083/TCP                     6m23s   app.kubernetes.io/name=argocd-server

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                         IMAGES                                             SELECTOR
deployment.apps/argocd-applicationset-controller   1/1     1            1           6m23s   argocd-applicationset-controller   quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-applicationset-controller
deployment.apps/argocd-dex-server                  1/1     1            1           6m23s   dex                                ghcr.io/dexidp/dex:v2.43.0                         app.kubernetes.io/name=argocd-dex-server
deployment.apps/argocd-notifications-controller    1/1     1            1           6m23s   argocd-notifications-controller    quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-notifications-controller
deployment.apps/argocd-redis                       1/1     1            1           6m23s   redis                              public.ecr.aws/docker/library/redis:7.2.7-alpine   app.kubernetes.io/name=argocd-redis
deployment.apps/argocd-repo-server                 1/1     1            1           6m23s   argocd-repo-server                 quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-repo-server
deployment.apps/argocd-server                      1/1     1            1           6m23s   argocd-server                      quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-server

NAME                                                          DESIRED   CURRENT   READY   AGE     CONTAINERS                         IMAGES                                             SELECTOR
replicaset.apps/argocd-applicationset-controller-578697b885   1         1         1       6m23s   argocd-applicationset-controller   quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-applicationset-controller,pod-template-hash=578697b885
replicaset.apps/argocd-dex-server-95477cdd                    1         1         1       6m23s   dex                                ghcr.io/dexidp/dex:v2.43.0                         app.kubernetes.io/name=argocd-dex-server,pod-template-hash=95477cdd
replicaset.apps/argocd-notifications-controller-787447c77d    1         1         1       6m23s   argocd-notifications-controller    quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-notifications-controller,pod-template-hash=787447c77d
replicaset.apps/argocd-redis-5746c4c5fb                       1         1         1       6m23s   redis                              public.ecr.aws/docker/library/redis:7.2.7-alpine   app.kubernetes.io/name=argocd-redis,pod-template-hash=5746c4c5fb
replicaset.apps/argocd-repo-server-588c6f4648                 1         1         1       6m23s   argocd-repo-server                 quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-repo-server,pod-template-hash=588c6f4648
replicaset.apps/argocd-server-656b9b6c6c                      1         1         1       6m23s   argocd-server                      quay.io/argoproj/argocd:v3.1.8                     app.kubernetes.io/name=argocd-server,pod-template-hash=656b9b6c6c

NAME                                             READY   AGE     CONTAINERS                      IMAGES
statefulset.apps/argocd-application-controller   1/1     6m23s   argocd-application-controller   quay.io/argoproj/argocd:v3.1.8
```

## 接続

ロードバランサに変更する。

```sh
kubectl -n argocd patch service argocd-server -p '{"spec": {"type": "LoadBalancer"}}'
```

```text
service/argocd-server patched
```

サービスを確認する。

```sh
kubectl -n argocd get service argocd-server
```

```text
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
argocd-server   LoadBalancer   10.100.127.147   172.16.0.100   80:32624/TCP,443:31163/TCP   8m26s
```

接続確認する。

```sh
curl -k -L https://172.16.0.100/
```

HTML が返却される。

## アカウントの作成

admin のパスワードを取得する。

```sh
argocd admin initial-password -n argocd
```

```text
9bX88h3mmRWxHpxv

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```

UI のログイン画面に入力する。
