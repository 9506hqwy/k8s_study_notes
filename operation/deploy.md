# デプロイ

ArgoCD を利用してデプロイする。

## リポジトリの準備

git リポジトリとして [gitweb](https://github.com/9506hqwy/container/tree/main/gitweb/ubuntu/image) をデプロイする。

コンテナイメージを作成する。

```sh
buildah bud -t registry.home.local/system/gitweb .
```

コンテナイメージをレジストリに登録する。

```sh
podman push registry.home.local/system/gitweb
```

デプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: git
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitweb
  namespace: git
spec:
  selector:
    matchLabels:
      app: git-server
  template:
    metadata:
      labels:
        app: git-server
    spec:
      containers:
      - name: gitweb
        image: registry.home.local/system/gitweb
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - mountPath: /mnt/repos
          name: repos
      volumes:
      - name: repos
        iscsi:
          targetPortal: 10.0.0.92:3260
          portals:
          - 0.0.0.0:3260
          iqn: iqn.2003-01.org.linux-iscsi.storage.x8664:sn.f64cd645a385
          lun: 1
          fsType: xfs
---
apiVersion: v1
kind: Service
metadata:
  name: gitweb
  namespace: git
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
  selector:
    app: git-server
  type: LoadBalancer
EOF
```

ポッドとサービスを確認する。

```sh
kubectl -n git get pod,svc -o wide
```

```
NAME                          READY   STATUS    RESTARTS   AGE     IP              NODE                  NOMINATED NODE   READINESS GATES
pod/gitweb-64c5d4df7b-k6vxt   1/1     Running   0          9m17s   172.17.51.139   worker02.home.local   <none>           <none>

NAME             TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE     SELECTOR
service/gitweb   LoadBalancer   10.98.242.215   172.16.0.101   80:30338/TCP   9m17s   app=git-server
```

リポジトリを作成する。

```sh
kubectl -n git exec -t gitweb-64c5d4df7b-k6vxt -- /bin/bash -c 'cd /mnt/repos && init_repo.sh manifest.git'
```

リポジトリをクローンする。

```sh
git clone http://172.16.0.101/gitrepo/manifest.git
```

```
Cloning into 'manifest'...
warning: You appear to have cloned an empty repository.
```

リポジトリにマニフェストファイルを追加する。

```yaml
# web/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      containers:
      - name: wen
        image: nginx
        ports:
        - containerPort: 80
          protocol: TCP
# web/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
  selector:
    app: web-server
  type: ClusterIP
# web/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  ingressClassName: nginx
  rules:
  - host: web.home.local
    http:
      paths:
      - backend:
          service:
            name: web
            port:
              number: 80
        path: /
        pathType: Prefix
```

## ログイン

ArgoCD にログインする。

```sh
argocd login --insecure 172.16.0.102
Username: admin
Password:
```

```
'admin:login' logged in successfully
Context '172.16.0.102' updated
```

## リポジトリの登録

ArgoCD にリポジトリを登録する。

```sh
argocd repo add http://172.16.0.101/gitrepo/manifest.git
```

```
Repository 'http://172.16.0.101/gitrepo/manifest.git' added
```

リポジトリを確認する。

```sh
argocd repo list
```

```
TYPE  NAME  REPO                                      INSECURE  OCI    LFS    CREDS  STATUS      MESSAGE  PROJECT
git         http://172.16.0.101/gitrepo/manifest.git  false     false  false  false  Successful
```

## アプリケーションの追加

ArgoCD にアプリケーションを追加する。

```sh
argocd app create web \
    --repo http://172.16.0.101/gitrepo/manifest.git \
    --sync-option CreateNamespace=true \
    --path web \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace argocd-sample
```

```
application 'web' created
```

アプリケーションを確認する。

```sh
argocd app list
```

```
NAME        CLUSTER                         NAMESPACE      PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                      PATH  TARGET
argocd/web  https://kubernetes.default.svc  argocd-sample  default  OutOfSync  Missing  <none>      <none>      http://172.16.0.101/gitrepo/manifest.git  web
```

## アプリケーションのデプロイ

アプリケーションをデプロイする。

```sh
argocd app sync web
```

```
IMESTAMP                  GROUP                    KIND   NAMESPACE                     NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-11-15T00:26:28+09:00                        Service  argocd-sample                   web  OutOfSync  Missing
2024-11-15T00:26:28+09:00   apps              Deployment  argocd-sample                   web  OutOfSync  Missing
2024-11-15T00:26:28+09:00  networking.k8s.io     Ingress  argocd-sample                   web  OutOfSync  Missing
2024-11-15T00:26:31+09:00          Namespace                     argocd-sample   Running   Synced              namespace/argocd-sample created
2024-11-15T00:26:31+09:00            Service  argocd-sample                   web    Synced  Healthy
2024-11-15T00:26:31+09:00   apps  Deployment  argocd-sample                   web    Synced  Progressing

Name:               argocd/web
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          argocd-sample
URL:                https://172.16.0.102/applications/web
Repo:               http://172.16.0.101/gitrepo/manifest.git
Target:
Path:               web
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (1211ed7)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      1211ed7816d96bd126347821407013cc2208f508
Phase:              Succeeded
Start:              2024-11-15 00:26:29 +0900 JST
Finished:           2024-11-15 00:26:31 +0900 JST
Duration:           2s
Message:            successfully synced (all tasks run)

GROUP              KIND        NAMESPACE      NAME           STATUS   HEALTH       HOOK  MESSAGE
                   Namespace                  argocd-sample  Running  Synced             namespace/argocd-sample created
                   Service     argocd-sample  web            Synced   Healthy            service/web created
apps               Deployment  argocd-sample  web            Synced   Progressing        deployment.apps/web created
networking.k8s.io  Ingress     argocd-sample  web            Synced   Progressing        ingress.networking.k8s.io/web created
```

アプリケーションのステータスを確認する。

```sh
argocd app get web
```

```
Name:               argocd/web
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          argocd-sample
URL:                https://172.16.0.102/applications/web
Repo:               http://172.16.0.101/gitrepo/manifest.git
Target:
Path:               web
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (1211ed7)
Health Status:      Healthy

GROUP              KIND        NAMESPACE      NAME           STATUS   HEALTH   HOOK  MESSAGE
                   Namespace                  argocd-sample  Running  Synced         namespace/argocd-sample created
                   Service     argocd-sample  web            Synced   Healthy        service/web created
apps               Deployment  argocd-sample  web            Synced   Healthy        deployment.apps/web created
networking.k8s.io  Ingress     argocd-sample  web            Synced   Healthy        ingress.networking.k8s.io/web created
```

リソースを確認する。

```sh
kubectl -n argocd-sample get all -o wide
```

```
NAME                       READY   STATUS    RESTARTS   AGE     IP              NODE                  NOMINATED NODE   READINESS GATES
pod/web-6fcfcf5cfb-kcg9c   1/1     Running   0          4m24s   172.17.51.132   worker02.home.local   <none>           <none>

NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE     SELECTOR
service/web   ClusterIP   10.101.227.188   <none>        80/TCP    4m24s   app=web-server

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
deployment.apps/web   1/1     1            1           4m24s   wen          nginx    app=web-server

NAME                             DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES   SELECTOR
replicaset.apps/web-6fcfcf5cfb   1         1         1       4m24s   wen          nginx    app=web-server,pod-template-hash=6fcfcf5cfb

NAME                            CLASS   HOSTS            ADDRESS        PORTS   AGE
ingress.networking.k8s.io/web   nginx   web.home.local   172.16.0.100   80      5m41s
```

接続を確認する。

```sh
curl -sSL --resolve web.home.local:80:172.16.0.100 http://web.home.local/
```

HTML が返却される。
