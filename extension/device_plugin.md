# デバイス・プラグイン

ワーカノード上のデバイスをコンテナで使用する仕組み。

ワーカノード上でデバイスを制御するサービスが必要となる。
動作確認に [sample-device](https://github.com/9506hqwy/k8s-device-plugin-rs) を使用する。

## プラグイン・サービス

プラグインサービスは、プラグインの登録、制御するデバイスの通知やデバイスの状態変更を監視する。

プラグインサービスは kubelet と UNIX ドメインソケットを使って通信する。

```sh
RUST_LOG=trace ./sample-device
```

```text
[2025-10-18T01:27:13Z TRACE device_plugin] Starting to listen
[2025-10-18T01:27:13Z TRACE device_plugin] Registration.register
[2025-10-18T01:27:13Z TRACE sample_device] DevicePlugin.get_device_plugin_options
[2025-10-18T01:27:13Z TRACE sample_device] DevicePlugin.list_and_watch
[2025-10-18T01:27:13Z TRACE sample_device] DevicePlugin.list_and_watch.start
[2025-10-18T01:27:13Z TRACE sample_device] DevicePlugin.list_and_watch.send
```

## ポッドのデプロイ

デバイス・プラグインが制御するデバイスを指定してポッドをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: demo-dp
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      limits:
        demo/sample-device: 2
EOF
```

```text
pod/demo-dp created
```

デバイス・プラグインの `allocate` が呼ばれる。

```text
[2025-10-18T01:28:11Z TRACE sample_device] DevicePlugin.allocate
```

ポッドを確認する。

```sh
kubectl get pods demo-dp -o wide
```

```text
NAME      READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
demo-dp   1/1     Running   0          34s   172.17.51.138   worker02.home.local   <none>           <none>
```

2 つのデバイスが割り当てられていることを確認する。

```sh
kubectl exec demo-dp -- env | grep SAMPLE
```

```text
SAMPLE_DEVICE1=1
SAMPLE_DEVICE4=1
```

## デバイスがない場合

デバイス・プラグインが提供している数以上を指定する。

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: demo-dp-overflow
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      limits:
        demo/sample-device: 5
EOF
```

```text
pod/demo-dp-overflow created
```

ポッドを確認する。

```sh
kubectl get pods demo-dp-overflow -o wide
```

```text
NAME               READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
demo-dp-overflow   0/1     Pending   0          11s   <none>   <none>   <none>           <none>
```

`Pending` のままリソース待ちになる。

## デバイス・プラグインが起動していない場合

デバイス・プラグインを停止してポッドをデプロイする。

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: demo-dp-not-service
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      limits:
        demo/sample-device: 1
EOF
```

ポッドを確認する。

```sh
kubectl get pod demo-dp-not-service -o wide
```

```text
NAME                  READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
demo-dp-not-service   0/1     Pending   0          20s   <none>   <none>   <none>           <none>
```

デバイス・プラグインを起動して待つ。

```text
NAME                  READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
demo-dp-not-service   1/1     Running   0          59s   172.17.51.139   worker02.home.local   <none>           <none>
```

## 参考

- [#3573 Add Device Manager to kubelet](https://github.com/kubernetes/enhancements/issues/3573)
- [Device Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)
