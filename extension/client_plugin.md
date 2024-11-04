# クライントプラグイン

`kubectl` コマンドを拡張する。

`kubectl-XXXX` の実行ファイルを環境変数 `PATH` の下に配置することで `kubectl XXXX` の形式で呼び出しができる。

動作確認用に [k8s-client-plugin-rs](https://github.com/9506hqwy/k8s-client-plugin-rs) を使用する。
[](./custom_controller.md) で管理するカスタムリソース `samples` を作成するコマンドを提供する。

## インストール

`kubectl-create-sample` を環境変数 `PATH` の下に配置する。

```sh
kubectl create sample -h
```

```
Usage: kubectl-create-sample [OPTIONS] --spec <SPEC_NAME> <NAME>

Arguments:
  <NAME>

Options:
  -n, --namespace <NAMESPACE>
  -s, --spec <SPEC_NAME>
  -h, --help                   Print help
  -V, --version                Print version
```

## カスタムリソースの作成

カスタムリソースを作成する。

```sh
kubectl create sample cr01 --spec s
```

```
sample.sample.custom-controller/cr01 created
```

カスタムリソースを確認する。

```sh
kubectl get sample cr01 -o yaml
```

```yaml
apiVersion: sample.custom-controller/v1alpha1
kind: Sample
metadata:
  creationTimestamp: "2024-11-04T10:41:35Z"
  finalizers:
  - sample-custom-controller/v1alpha1
  generation: 1
  name: cr01
  namespace: default
  resourceVersion: "576152"
  uid: 4113194e-2f86-4447-aebf-9bdfd1f843fa
spec:
  name: s
status:
  check: true
```

## カスタムリソースの削除

カスタムリソースを削除する。

```sh
kubectl delete sample cr01
```

```
sample.sample.custom-controller "cr01" deleted
```
