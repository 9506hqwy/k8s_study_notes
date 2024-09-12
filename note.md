# 注記

## トークンの期限切れ

ワーカノードを追加するときにエラーが発生する。

```sh
:linenos:
kubeadm join ...
```

```console
:linenos:
error execution phase preflight: couldn't validate the identity of the API Server: failed to request the cluster-info ConfigMap: client rate limiter Wait returned an error: context deadline exceeded
To see the stack trace of this error execute with --v=5 or higher
```

トークンを再作成する。

```sh
:linenos:
kubeadm token create
```

```console
:linenos:
xgii85.xkqpzwfrtq2urpt0
```
