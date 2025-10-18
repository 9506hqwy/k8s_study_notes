# 拡張 API サーバ

Kubernetes API Server を拡張してカスタムリソースを制御する仕組み。

クラスタ内で Service として作成し APIService リソースを登録する必要がある。
動作確認に [sample-api-server](https://github.com/9506hqwy/k8s-api-server-rs) を使用する。

## 拡張 API サーバのコンテナイメージ

拡張 API サーバをコンテナイメージとしてコンテナレジストリに登録する。

```sh
buildah bud -t registry.home.local/system/sample-api-server -f sample-api-server/Dockerfile .
```

```sh
podman push registry.home.local/system/sample-api-server
```

## サービスの登録

名前空間を作成する。

```sh
kubectl create namespace sample-system
```

```text
namespace/sample-system created
```

サービスアカウントを作成する。

```sh
kubectl create serviceaccount sample-api-server -n sample-system
```

```text
serviceaccount/sample-api-server created
```

サービスアカウントにクラスタロール `system:auth-delegator` を割り当てる。

```sh
kubectl create clusterrolebinding sample-api-server-auth \
    --clusterrole=system:auth-delegator \
    --serviceaccount=sample-system:sample-api-server
```

```text
clusterrolebinding.rbac.authorization.k8s.io/sample-api-server-auth created
```

サービスアカウントにロール `extension-apiserver-authentication-reader` を割り当てる。

```sh
kubectl create rolebinding sample-api-server-config-reader \
    -n kube-system \
    --role=extension-apiserver-authentication-reader \
    --serviceaccount=sample-system:sample-api-server
```

```text
rolebinding.rbac.authorization.k8s.io/sample-api-server-config-reader created
```

ポッドを作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-api-server
  namespace: sample-system
spec:
  selector:
    matchLabels:
      app: sample-api-server
  template:
    metadata:
      labels:
        app: sample-api-server
    spec:
      serviceAccountName: sample-api-server
      containers:
      - name: sample-api-server
        image: registry.home.local/system/sample-api-server
        env:
        - name: RUST_LOG
          value: debug
        ports:
        - containerPort: 3000
          protocol: TCP
EOF
```

```text
deployment.apps/sample-api-server created
```

ポッドを確認する。

```sh
kubectl -n sample-system get pod -o wide
```

```text
NAME                                 READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
sample-api-server-57cf8c89dd-9wxch   1/1     Running   0          16s   172.17.255.150   worker01.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl -n sample-system expose deployment sample-api-server
```

```text
service/sample-api-server exposed
```

サービスを確認する。

```sh
kubectl -n sample-system get service sample-api-server -o wide
```

```text
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
sample-api-server   ClusterIP   10.108.235.177   <none>        3000/TCP   16s   app=sample-api-server
```

## ログレベルを変更

Kubernetes API Server のログレベルを変更する。

トークンを作成する。

```sh
kubectl create token default
```

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IlM...
```

ログレベルを変更する。

```sh
curl -k -H "Authorization: Bearer $TOKEN" -d 5 -X PUT  https://controller.home.local:6443/debug/flags/v
```

```text
successfully set klog.logging.verbosity to 5
```

## APIService の登録

サービスを APIService として登録する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1alpha1.sample-api-server
spec:
  groupPriorityMinimum: 3000
  versionPriority: 10
  group: sample-api-server
  insecureSkipTLSVerify: true
  service:
    name: sample-api-server
    namespace: sample-system
    port: 3000
  version: v1alpha1
EOF
```

```text
apiservice.apiregistration.k8s.io/v1alpha1.sample-api-server created
```

APIService を確認する。

```sh
kubectl get apiservice v1alpha1.sample-api-server
```

```text
NAME                         SERVICE                           AVAILABLE   AGE
v1alpha1.sample-api-server   sample-system/sample-api-server   True        14s
```

## 動作確認

APIResource を確認する。

```sh
kubectl get --raw '/apis/sample-api-server/v1alpha1' | jq
```

```json
{
  "apiVersion": "v1",
  "kind": "APIResourceList",
  "groupVersion": "sample-api-server/v1alpha1",
  "resources": [
    {
      "group": "sample-api-server",
      "kind": "Sample",
      "name": "samples",
      "namespaced": true,
      "singularName": "sample",
      "verbs": [
        "get",
        "list"
      ]
    }
  ]
}
```

`list` を確認する。

```sh
kubectl get --raw '/apis/sample-api-server/v1alpha1/namespaces/default/samples' | jq
```

```json
{
  "apiVersion": "sample-api-server/v1alpha1",
  "kind": "SampleList",
  "metadata": {
    "resourceVersion": "1"
  },
  "items": [
    {
      "apiVersion": "sample-api-server/v1alpha1",
      "kind": "Sample",
      "metadata": {
        "name": "sample1"
      },
      "spec": {
        "name": "default"
      }
    }
  ]
}
```

`get` を確認する。

```sh
kubectl get --raw '/apis/sample-api-server/v1alpha1/namespaces/default/samples/a' | jq
```

```json
{
  "apiVersion": "sample-api-server/v1alpha1",
  "kind": "Sample",
  "metadata": {
    "name": "a"
  },
  "spec": {
    "name": "default.a"
  }
}
```

リソース一覧を取得する。

```sh
kubectl get samples
```

```text
NAME      AGE
sample1   <unknown>
```

リソースを取得する。

```sh
kubectl get samples a
```

```text
NAME   AGE
a      <unknown>
```

## アグリゲーションレイヤーの起動

```{note}
エントリポイントはあっている？
```

アグリゲーションレイヤーの起動時は以下の処理が行われる。

1. [RunAggregator](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/cmd/server/start.go#L128-L181)
   1. [NewWithDelegate](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiserver.go#L205-L461)
      - [APIServiceRegistrationController](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiservice_controller.go) の登録
        - APIService リソースの登録や解除を実行する。
      - [Local AvailableConditionController](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/local/local_available_controller.go) の登録
        - Service リソースと関連しない APIService リソースのステータスを更新する。
      - [Remote AvailableConditionController](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/remote/remote_available_controller.go) の登録
        - Service リソースと関連する APIService リソースのステータスを更新する。
      - [DiscoveryAggregationController](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/handler_discovery.go) の登録
        - APIService リソースと APIResource リソースのマッピングをキャッシュする。
      - 他、省略。

## APIServiceRegistrationController

APIService リソースの登録と解除を実行する。

APIServiceRegistrationController は起動時に APIService informer にあるリソースをすべて API Handler に登録する。

1. [Run](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiservice_controller.go#L109-L120)
   1. [AddAPIService](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiserver.go#L521-L596)

APIService の更新があると登録と解除が行われる。

1. [sync](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiservice_controller.go#L82-L93)
   1. [AddAPIService](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiserver.go#L521-L596)
   2. [RemoveAPIService](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/apiserver.go#L600-L635)

## Remote AvailableConditionController

Service リソースに接続して APIService リソースのステータスを更新する。

1. APIService リソースに設定されている Service リソースが存在しない場合は [ServiceNotFound](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/remote/remote_available_controller.go#L205-L211)
2. Endpoint リソースが存在しない場合は [EndpointsNotFound](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/remote/remote_available_controller.go#L243-L249)
3. 有効な Endpoint リソースが存在しない場合は [MissingEndpoints](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/remote/remote_available_controller.go#L271-L277)
4. */apis/\<GROUP>/\<VERSION>* への HTTP GET が失敗する場合は [FailedDiscoveryCheck](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/controllers/status/remote/remote_available_controller.go#L349-L360)

## DiscoveryAggregationController

*/apis/\<GROUP>/\<VERSION>* から APIResouce リソースを取得して [キャッシュ](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/apiserver/handler_discovery.go#L315-L376) する。

## リクエストの認証

拡張 API サーバ起動時に認証設定を行う。

1. [DefaultBuildHandlerChain](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/config.go#L1029)
   1. [withAuthentication](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/endpoints/filters/authentication.go#L60-L121)
      - リクエスト受信時に `c.Authentication.Authenticator.AuthenticateRequest` が実行される。

アグリゲーションレイヤーの起動時に認証設定を行う。

1. [NewDefaultOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/cmd/server/start.go#L100)
   1. [NewRecommendedOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/options/recommended.go#L67)
      1. [DelegatingAuthenticationOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/options/authentication.go#L388-L392)
         1. [DelegatingAuthenticatorConfig](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authentication/authenticatorfactory/delegating.go#L76-L82)
            1. [NewDynamicVerifyOptionsSecure](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authentication/request/headerrequest/requestheader.go#L106-L109)
               1. [NewDynamicCAVerifier](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authentication/request/x509/x509.go#L220-L222)
                  - クライアント証明書認証が設定される。

リクエスト受信時にクライアント証明書の CN を確認する。

1. [verifySubject](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authentication/request/x509/x509.go#L246-L248)

## リクエストの認可

拡張 API サーバ起動時に認可設定を行う。

1. [DefaultBuildHandlerChain](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/config.go#L1002)
   1. [withAuthorization](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/endpoints/filters/authorization.go#L62-L97)
      - リクエスト受信時に `c.Authorization.Authorizer` が実行される。

アグリゲーションレイヤーの起動時に認証設定を行う。

1. [NewDefaultOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/kube-aggregator/pkg/cmd/server/start.go#L100)
   1. [NewRecommendedOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/options/recommended.go#L68)
      1. [DelegatingAuthorizationOptions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/server/options/authorization.go#L173)
         1. [NewPrivilegedGroups](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authorization/authorizerfactory/builtin.go#L91-L95)
            - グループを確認する。
         2. [NewAuthorizer](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authorization/path/path.go#L50-L67)
            - リソースリクエストではない URI を確認する。
         3. [DelegatingAuthenticatorConfig](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/pkg/authorization/authorizerfactory/delegating.go#L52-L59)
            1. [newWithBackoff](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go#L128-L138)
               - SubjectAccessReview を API Server にリクエストする。

リクエスト受信時にヘッダに設定されたユーザ名とグループから SubjectAccessReview を実行する。

1. [Authorize](https://github.com/kubernetes/kubernetes/blob/v1.31.1/staging/src/k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go#L240)
