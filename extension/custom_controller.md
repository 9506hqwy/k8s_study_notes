# カスタムコントローラ

カスタムリソースを監視し、制御する仕組み。

[](./custom_resource_definition.md) と組み合わせて、カスタムリソースを管理する仕組みがオペレータと呼ばれる。

Kuberntes API Server と通信してリソースを監視し変化があれば処理を実行する。
動作確認に [sample-custom-controller](https://github.com/9506hqwy/k8s-custom-controller-rs) を使用する。

カスタムコントローラを Deployment としてデプロイする。

## カスタムコントローラのコンテナイメージ

拡張 API サーバをコンテナイメージとしてコンテナレジストリに登録する。

```sh
buildah bud --format=docker -t registry.home.local/system/sample-custom-controller -f sample-custom-controller/Dockerfile .
```

```sh
podman push registry.home.local/system/sample-custom-controller
```

## カスタムリソース定義の登録

```sh
crdgen | kubectl apply -f -
```

```
customresourcedefinition.apiextensions.k8s.io/samples.sample.custom-controller created
```

## カスタムコントローラの登録

名前空間を作成する。

```sh
kubectl create namespace sample-system
```

```
namespace/sample-system created
```

サービスアカウントを作成する。

```sh
kubectl create serviceaccount sample-custom-controller -n sample-system
```

```
serviceaccount/sample-custom-controller created
```

ClusterRole を作成する。

```sh
kubectl create clusterrole sample-custom-controller \
    --resource="samples,samples/status" \
    --verb="get,list,patch,watch"
```

```
clusterrole.rbac.authorization.k8s.io/sample-custom-controller created
```

サービスアカウントに ClusterRole を割り当てる。

```sh
kubectl create clusterrolebinding sample-custom-controller \
    --clusterrole=sample-custom-controller \
    --serviceaccount=sample-system:sample-custom-controller
```

```
clusterrolebinding.rbac.authorization.k8s.io/sample-custom-controller created
```

Deployment を作成する。

```yaml
# deployment-sample-custom-controller.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-custom-controller
  namespace: sample-system
spec:
  selector:
    matchLabels:
      app: sample-custom-controller
  template:
    metadata:
      labels:
        app: sample-custom-controller
    spec:
      serviceAccountName: sample-custom-controller
      containers:
      - name: sample-custom-controller
        image: registry.home.local/system/sample-custom-controller
        env:
        - name: RUST_LOG
          value: info
```

```sh
kubectl apply -f deployment-sample-custom-controller.yaml
```

```
deployment.apps/sample-custom-controller created
```

ポッドを確認する。

```sh
kubectl -n sample-system get pod -o wide
```

```
NAME                                        READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
sample-custom-controller-68855d5c9c-k88nn   1/1     Running   0          26s   172.17.51.154   worker02.home.local   <none>           <none>
```

## 動作確認

リソースを登録する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: sample.custom-controller/v1alpha1
kind: Sample
metadata:
  name: custom-sample-cr-01
  namespace: default
spec:
  name: sample-01
EOF
```

```
sample.sample.custom-controller/custom-sample-cr-01 created
```

リソースを確認する。`status.check` がコントローラで更新されている。

```sh
kubectl get samples custom-sample-cr-01 -o yaml
```

```yaml
apiVersion: sample.custom-controller/v1alpha1
kind: Sample
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"sample.custom-controller/v1alpha1","kind":"Sample","metadata":{"annotations":{},"name":"custom-sample-cr-01","namespace":"default"},"spec":{"name":"sample-01"}}
  creationTimestamp: "2024-10-27T10:33:30Z"
  finalizers:
  - sample-custom-controller/v1alpha1
  generation: 1
  name: custom-sample-cr-01
  namespace: default
  resourceVersion: "542107"
  uid: 72d643c8-4d71-43d1-93b7-0baac3e12ab4
spec:
  name: sample-01
status:
  check: true
```

コントローラに以下のログが出力される。3 回リソースの変更が呼ばれる。

```
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Start custom-sample-cr-01
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Ok custom-sample-cr-01 Action { requeue_after: None }
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Start custom-sample-cr-01
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Apply custom-sample-cr-01
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Ok custom-sample-cr-01 Action { requeue_after: None }
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Start custom-sample-cr-01
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Apply custom-sample-cr-01
[2024-10-27T10:33:30Z INFO  sample_custom_controller] Ok custom-sample-cr-01 Action { requeue_after: None }
```

リソースを削除する。

```sh
kubectl delete samples custom-sample-cr-01
```

```
sample.sample.custom-controller "custom-sample-cr-01" deleted
```

コントローラに以下のログが出力される。1 回リソースの変更が呼ばれる。

```
[2024-10-27T10:45:30Z INFO  sample_custom_controller] Start custom-sample-cr-01
[2024-10-27T10:45:30Z INFO  sample_custom_controller] Cleanup custom-sample-cr-01
[2024-10-27T10:45:30Z INFO  sample_custom_controller] Ok custom-sample-cr-01 Action { requeue_after: None }
```
