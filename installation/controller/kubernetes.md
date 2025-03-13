# Kubernetes

## コントロールプレーン

コントロールプレーンに Pod をスケジュールリングする。

```sh
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

```text
node/controller.home.local untainted
```
