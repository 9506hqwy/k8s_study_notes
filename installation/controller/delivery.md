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
argocd-application-controller-0                     1/1     Running   0          66s
argocd-applicationset-controller-5b866bf4f7-qzvqt   1/1     Running   0          67s
argocd-dex-server-7b6987df7-lf9bs                   1/1     Running   0          67s
argocd-notifications-controller-5ddc4fdfb9-vhqn4    1/1     Running   0          67s
argocd-redis-ffccd77b9-x9t7f                        1/1     Running   0          67s
argocd-repo-server-55bb7b784-l26xs                  1/1     Running   0          67s
argocd-server-7c746df554-6zf9h                      1/1     Running   0          66s
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
NAME                                                    READY   STATUS    RESTARTS   AGE     IP               NODE                  NOMINATED NODE   READINESS GATES
pod/argocd-application-controller-0                     1/1     Running   0          2m38s   172.17.255.159   worker01.home.local   <none>           <none>
pod/argocd-applicationset-controller-5b866bf4f7-qzvqt   1/1     Running   0          2m39s   172.17.51.154    worker02.home.local   <none>           <none>
pod/argocd-dex-server-7b6987df7-lf9bs                   1/1     Running   0          2m39s   172.17.255.160   worker01.home.local   <none>           <none>
pod/argocd-notifications-controller-5ddc4fdfb9-vhqn4    1/1     Running   0          2m39s   172.17.51.155    worker02.home.local   <none>           <none>
pod/argocd-redis-ffccd77b9-x9t7f                        1/1     Running   0          2m39s   172.17.255.157   worker01.home.local   <none>           <none>
pod/argocd-repo-server-55bb7b784-l26xs                  1/1     Running   0          2m39s   172.17.255.158   worker01.home.local   <none>           <none>
pod/argocd-server-7c746df554-6zf9h                      1/1     Running   0          2m38s   172.17.51.156    worker02.home.local   <none>           <none>

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE     SELECTOR
service/argocd-applicationset-controller          ClusterIP   10.105.254.61    <none>        7000/TCP,8080/TCP            2m39s   app.kubernetes.io/name=argocd-applicationset-controller
service/argocd-dex-server                         ClusterIP   10.103.47.213    <none>        5556/TCP,5557/TCP,5558/TCP   2m39s   app.kubernetes.io/name=argocd-dex-server
service/argocd-metrics                            ClusterIP   10.109.234.178   <none>        8082/TCP                     2m39s   app.kubernetes.io/name=argocd-application-controller
service/argocd-notifications-controller-metrics   ClusterIP   10.106.252.108   <none>        9001/TCP                     2m39s   app.kubernetes.io/name=argocd-notifications-controller
service/argocd-redis                              ClusterIP   10.108.57.155    <none>        6379/TCP                     2m39s   app.kubernetes.io/name=argocd-redis
service/argocd-repo-server                        ClusterIP   10.98.136.128    <none>        8081/TCP,8084/TCP            2m39s   app.kubernetes.io/name=argocd-repo-server
service/argocd-server                             ClusterIP   10.101.28.50     <none>        80/TCP,443/TCP               2m39s   app.kubernetes.io/name=argocd-server
service/argocd-server-metrics                     ClusterIP   10.108.154.59    <none>        8083/TCP                     2m39s   app.kubernetes.io/name=argocd-server

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                         IMAGES                            SELECTOR
deployment.apps/argocd-applicationset-controller   1/1     1            1           2m39s   argocd-applicationset-controller   quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-applicationset-controller
deployment.apps/argocd-dex-server                  1/1     1            1           2m39s   dex                                ghcr.io/dexidp/dex:v2.38.0        app.kubernetes.io/name=argocd-dex-server
deployment.apps/argocd-notifications-controller    1/1     1            1           2m39s   argocd-notifications-controller    quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-notifications-controller
deployment.apps/argocd-redis                       1/1     1            1           2m39s   redis                              redis:7.0.15-alpine               app.kubernetes.io/name=argocd-redis
deployment.apps/argocd-repo-server                 1/1     1            1           2m39s   argocd-repo-server                 quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-repo-server
deployment.apps/argocd-server                      1/1     1            1           2m38s   argocd-server                      quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-server

NAME                                                          DESIRED   CURRENT   READY   AGE     CONTAINERS                         IMAGES                            SELECTOR
replicaset.apps/argocd-applicationset-controller-5b866bf4f7   1         1         1       2m39s   argocd-applicationset-controller   quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-applicationset-controller,pod-template-hash=5b866bf4f7
replicaset.apps/argocd-dex-server-7b6987df7                   1         1         1       2m39s   dex                                ghcr.io/dexidp/dex:v2.38.0        app.kubernetes.io/name=argocd-dex-server,pod-template-hash=7b6987df7
replicaset.apps/argocd-notifications-controller-5ddc4fdfb9    1         1         1       2m39s   argocd-notifications-controller    quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-notifications-controller,pod-template-hash=5ddc4fdfb9
replicaset.apps/argocd-redis-ffccd77b9                        1         1         1       2m39s   redis                              redis:7.0.15-alpine               app.kubernetes.io/name=argocd-redis,pod-template-hash=ffccd77b9
replicaset.apps/argocd-repo-server-55bb7b784                  1         1         1       2m39s   argocd-repo-server                 quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-repo-server,pod-template-hash=55bb7b784
replicaset.apps/argocd-server-7c746df554                      1         1         1       2m38s   argocd-server                      quay.io/argoproj/argocd:v2.12.3   app.kubernetes.io/name=argocd-server,pod-template-hash=7c746df554

NAME                                             READY   AGE     CONTAINERS                      IMAGES
statefulset.apps/argocd-application-controller   1/1     2m38s   argocd-application-controller   quay.io/argoproj/argocd:v2.12.3
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
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
argocd-server   LoadBalancer   10.101.28.50   172.16.0.102   80:31080/TCP,443:30651/TCP   10m
```

接続確認する。

```sh
curl -k -L https://172.16.0.102/
```

HTML が返却される。

## アカウントの作成

admin のパスワードを取得する。

```sh
argocd admin initial-password -n argocd
```

```text
7UGCbCAhWFRPi0N7

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```

UI のログイン画面に入力する。
