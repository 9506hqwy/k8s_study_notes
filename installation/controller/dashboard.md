# ダッシュボード

## Helm

tar をインストールする。

```sh
dnf install -y tar
```

Helm をインストールする。

```sh
mkdir -p $HOME/.local/bin
curl -fsSL -o - https://get.helm.sh/helm-v3.19.0-linux-amd64.tar.gz | \
    tar --strip-components=1 -zxf - -O linux-amd64/helm > $HOME/.local/bin/helm
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
LAST DEPLOYED: Sun Oct 12 20:57:40 2025
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
NAME                                                   READY   STATUS    RESTARTS   AGE
kubernetes-dashboard-api-6d5c65b695-8vf8x              1/1     Running   0          24s
kubernetes-dashboard-auth-d5bb88695-vxxvs              1/1     Running   0          24s
kubernetes-dashboard-kong-648658d45f-h8smk             1/1     Running   0          24s
kubernetes-dashboard-metrics-scraper-547874fcf-lmmrr   1/1     Running   0          24s
kubernetes-dashboard-web-7796b9fbbb-48cl7              1/1     Running   0          24s
```

## 環境確認

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n kubernetes-dashboard -o wide
```

```text
NAME                                                       READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
pod/kubernetes-dashboard-api-6d5c65b695-8vf8x              1/1     Running   0          38s   172.17.2.36   controller.home.local   <none>           <none>
pod/kubernetes-dashboard-auth-d5bb88695-vxxvs              1/1     Running   0          38s   172.17.2.39   controller.home.local   <none>           <none>
pod/kubernetes-dashboard-kong-648658d45f-h8smk             1/1     Running   0          38s   172.17.2.35   controller.home.local   <none>           <none>
pod/kubernetes-dashboard-metrics-scraper-547874fcf-lmmrr   1/1     Running   0          38s   172.17.2.37   controller.home.local   <none>           <none>
pod/kubernetes-dashboard-web-7796b9fbbb-48cl7              1/1     Running   0          38s   172.17.2.38   controller.home.local   <none>           <none>

NAME                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
service/kubernetes-dashboard-api               ClusterIP   10.96.228.71     <none>        8000/TCP   38s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-auth              ClusterIP   10.102.152.48    <none>        8000/TCP   38s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-kong-proxy        ClusterIP   10.109.222.188   <none>        443/TCP    38s   app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong
service/kubernetes-dashboard-metrics-scraper   ClusterIP   10.106.177.103   <none>        8000/TCP   38s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard
service/kubernetes-dashboard-web               ClusterIP   10.105.186.19    <none>        8000/TCP   38s   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard

NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                             IMAGES                                                   SELECTOR
deployment.apps/kubernetes-dashboard-api               1/1     1            1           38s   kubernetes-dashboard-api               docker.io/kubernetesui/dashboard-api:1.13.0              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-auth              1/1     1            1           38s   kubernetes-dashboard-auth              docker.io/kubernetesui/dashboard-auth:1.3.0              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-kong              1/1     1            1           38s   proxy                                  kong:3.8                                                 app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong
deployment.apps/kubernetes-dashboard-metrics-scraper   1/1     1            1           38s   kubernetes-dashboard-metrics-scraper   docker.io/kubernetesui/dashboard-metrics-scraper:1.2.2   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard
deployment.apps/kubernetes-dashboard-web               1/1     1            1           38s   kubernetes-dashboard-web               docker.io/kubernetesui/dashboard-web:1.7.0               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard

NAME                                                             DESIRED   CURRENT   READY   AGE   CONTAINERS                             IMAGES                                                   SELECTOR
replicaset.apps/kubernetes-dashboard-api-6d5c65b695              1         1         1       38s   kubernetes-dashboard-api               docker.io/kubernetesui/dashboard-api:1.13.0              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-api,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=6d5c65b695
replicaset.apps/kubernetes-dashboard-auth-d5bb88695              1         1         1       38s   kubernetes-dashboard-auth              docker.io/kubernetesui/dashboard-auth:1.3.0              app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-auth,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=d5bb88695
replicaset.apps/kubernetes-dashboard-kong-648658d45f             1         1         1       38s   proxy                                  kong:3.8                                                 app.kubernetes.io/component=app,app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kong,pod-template-hash=648658d45f
replicaset.apps/kubernetes-dashboard-metrics-scraper-547874fcf   1         1         1       38s   kubernetes-dashboard-metrics-scraper   docker.io/kubernetesui/dashboard-metrics-scraper:1.2.2   app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-metrics-scraper,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=547874fcf
replicaset.apps/kubernetes-dashboard-web-7796b9fbbb              1         1         1       38s   kubernetes-dashboard-web               docker.io/kubernetesui/dashboard-web:1.7.0               app.kubernetes.io/instance=kubernetes-dashboard,app.kubernetes.io/name=kubernetes-dashboard-web,app.kubernetes.io/part-of=kubernetes-dashboard,pod-template-hash=7796b9fbbb
```

## 動作確認

フォワーディングする。

```sh
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

```text
Forwarding from 127.0.0.1:8443 -> 8443
Forwarding from [::1]:8443 -> 8443
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
kubernetes-dashboard   nginx   kubernetes-dashboard   172.16.0.102   80      98s
```

接続確認する。

```sh
curl -k --resolve kubernetes-dashboard:443:172.16.0.102 https://kubernetes-dashboard:443/
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
