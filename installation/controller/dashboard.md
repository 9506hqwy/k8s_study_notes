# ダッシュボード

## Helm

Helm をインストールする。

```sh
mkdir -p $HOME/.local/bin
curl -OL https://get.helm.sh/helm-v3.16.1-linux-amd64.tar.gz --output-dir /tmp
tar -C $HOME/.local/bin --strip-components=1 -zxf /tmp/helm-v3.16.1-linux-amd64.tar.gz linux-amd64/helm
chmod +x $HOME/.local/bin/helm
```

## インストール

リポジトリを追加する。

```sh
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

```text
"kubernetes-dashboard" has been added to your repositories
```

ダッシュボードを構築する。

```sh
helm upgrade --install \
    kubernetes-dashboard \
    kubernetes-dashboard/kubernetes-dashboard \
    --create-namespace \
    --namespace kubernetes-dashboard
```

```text
Release "kubernetes-dashboard" does not exist. Installing it now.
NAME: kubernetes-dashboard
LAST DEPLOYED: Mon Sep 30 12:08:53 2024
NAMESPACE: kubernetes-dashboard
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*************************************************************************************************
*** PLEASE BE PATIENT: Kubernetes Dashboard may need a few minutes to get up and become ready ***
*************************************************************************************************

Congratulations! You have just installed Kubernetes Dashboard in your cluster.

To access Dashboard run:
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

NOTE: In case port-forward command does not work, make sure that kong service name is correct.
      Check the services in Kubernetes Dashboard namespace using:
        kubectl -n kubernetes-dashboard get svc

Dashboard will be available at:
  https://localhost:8443
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pod -n kubernetes-dashboard
```

```text
NAME                                                    READY   STATUS    RESTARTS   AGE
kubernetes-dashboard-api-7d9b947756-swswq               1/1     Running   0          86s
kubernetes-dashboard-auth-648c6bdc7f-4fsg4              1/1     Running   0          86s
kubernetes-dashboard-kong-57d45c4f69-fblz2              1/1     Running   0          86s
kubernetes-dashboard-metrics-scraper-6b6f6f5d5c-7xj9s   1/1     Running   0          86s
kubernetes-dashboard-web-75cccd6488-tshj2               1/1     Running   0          86s
```

## 環境確認

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n kubernetes-dashboard -o wide
```

```text
NAME                                                        READY   STATUS    RESTARTS   AGE     IP               NODE                  NOMINATED NODE   READINESS GATES
pod/kubernetes-dashboard-api-7d9b947756-swswq               1/1     Running   0          5m44s   172.17.51.142    worker02.home.local   <none>           <none>
pod/kubernetes-dashboard-auth-648c6bdc7f-4fsg4              1/1     Running   0          5m44s   172.17.255.144   worker01.home.local   <none>           <none>
pod/kubernetes-dashboard-kong-57d45c4f69-fblz2              1/1     Running   0          5m44s   172.17.51.143    worker02.home.local   <none>           <none>
pod/kubernetes-dashboard-metrics-scraper-6b6f6f5d5c-7xj9s   1/1     Running   0          5m44s   172.17.255.143   worker01.home.local   <none>           <none>
pod/kubernetes-dashboard-web-75cccd6488-tshj2               1/1     Running   0          5m44s   172.17.51.141    worker02.home.local   <none>           <none>

NAME                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE     SELECTOR
service/kubernetes-dashboard-api               ClusterIP   10.111.38.67     <none>        8000/TCP                        5m45s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-auth              ClusterIP   10.110.49.5      <none>        8000/TCP                        5m45s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-kong-manager      NodePort    10.102.37.198    <none>        8002:30202/TCP,8445:31567/TCP   5m45s   app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong
service/kubernetes-dashboard-kong-proxy        ClusterIP   10.105.82.229    <none>        443/TCP                         5m45s   app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong
service/kubernetes-dashboard-metrics-scraper   ClusterIP   10.96.110.79     <none>        8000/TCP                        5m45s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-web               ClusterIP   10.100.163.120   <none>        8000/TCP                        5m45s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard

NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                             IMAGES                                                   SELECTOR
deployment.apps/kubernetes-dashboard-api               1/1     1            1           5m44s   kubernetes-dashboard-api               docker.io/kubernetesui/dashboard-api:1.8.1               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-auth              1/1     1            1           5m44s   kubernetes-dashboard-auth              docker.io/kubernetesui/dashboard-auth:1.1.3              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-kong              1/1     1            1           5m44s   proxy                                  kong:3.6                                                 app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong
deployment.apps/kubernetes-dashboard-metrics-scraper   1/1     1            1           5m44s   kubernetes-dashboard-metrics-scraper   docker.io/kubernetesui/dashboard-metrics-scraper:1.1.1   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-web               1/1     1            1           5m44s   kubernetes-dashboard-web               docker.io/kubernetesui/dashboard-web:1.4.0               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard

NAME                                                              DESIRED   CURRENT   READY   AGE     CONTAINERS                             IMAGES                                                   SELECTOR
replicaset.apps/kubernetes-dashboard-api-7d9b947756               1         1         1       5m44s   kubernetes-dashboard-api               docker.io/kubernetesui/dashboard-api:1.8.1               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=7d9b947756
replicaset.apps/kubernetes-dashboard-auth-648c6bdc7f              1         1         1       5m44s   kubernetes-dashboard-auth              docker.io/kubernetesui/dashboard-auth:1.1.3              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=648c6bdc7f
replicaset.apps/kubernetes-dashboard-kong-57d45c4f69              1         1         1       5m44s   proxy                                  kong:3.6                                                 app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong,pod-template-hash=57d45c4f69
replicaset.apps/kubernetes-dashboard-metrics-scraper-6b6f6f5d5c   1         1         1       5m44s   kubernetes-dashboard-metrics-scraper   docker.io/kubernetesui/dashboard-metrics-scraper:1.1.1   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=6b6f6f5d5c
replicaset.apps/kubernetes-dashboard-web-75cccd6488               1         1         1       5m44s   kubernetes-dashboard-web               docker.io/kubernetesui/dashboard-web:1.4.0               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=75cccd6488
```

## 動作確認

フォワーディングする。

```sh
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

```text
Forwarding from 127.0.0.1:8443 -> 8443
```

接続確認する。

```sh
curl -k https://127.0.0.1:8443/
```

HTML が返却される。

## Ingress

ingress を作成する。HTTPS をパススルーする。

```sh
kubectl -n kubernetes-dashboard create ingress \
    kubernetes-dashboard \
    --class=nginx \
    --annotation="nginx.ingress.kubernetes.io/backend-protocol=https" \
    --annotation="nginx.ingress.kubernetes.io/ssl-passthrough=true" \
    --rule="kubernetes-dashboard/*=kubernetes-dashboard-kong-proxy:443"
```

```text
ingress.networking.k8s.io/kubernetes-dashboard created
```

ingress を確認する。

```sh
kubectl -n kubernetes-dashboard get ingress
```

```text
NAME                   CLASS   HOSTS                  ADDRESS        PORTS   AGE
kubernetes-dashboard   nginx   kubernetes-dashboard   172.16.0.100   80      48s
```

接続確認する。

```sh
curl -k --resolve kubernetes-dashboard:443:172.16.0.100 https://kubernetes-dashboard:443/
```

HTML が返却される。

## アカウントの作成

サービスアカウントを作成する。

```sh
kubectl -n kubernetes-dashboard create serviceaccount admin-user
```

```text
serviceaccount/admin-user created
```

サービスアカウントとロールを紐づける。

```sh
kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
```

```text
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

## トークンの発行

サービスアカウントのトークンを作成する。

```sh
kubectl -n kubernetes-dashboard create token admin-user
```

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IlM4T...
```

ダッシュボードのログイン画面に入力する。
