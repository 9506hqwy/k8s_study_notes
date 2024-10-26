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
```
[2024-10-12T02:47:31Z TRACE device_plugin] Starting to listen
[2024-10-12T02:47:31Z TRACE device_plugin] Registration.register
[2024-10-12T02:47:31Z TRACE sample_device] DevicePlugin.get_device_plugin_options
[2024-10-12T02:47:31Z TRACE sample_device] DevicePlugin.list_and_watch
[2024-10-12T02:47:31Z TRACE sample_device] DevicePlugin.list_and_watch.start
[2024-10-12T02:47:31Z TRACE sample_device] DevicePlugin.list_and_watch.send
```

## ポッドのデプロイ

デバイス・プラグインが制御するデバイスを指定してポッドをデプロイする。

```yaml
# pod.yaml
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
```

デプロイする。

```sh
kubectl apply -f pod.yaml
```

```
pod/demo-dp created
```

デバイス・プラグインの `allocate` が呼ばれる。

```
[2024-10-12T04:10:49Z TRACE sample_device] DevicePlugin.allocate
```

ポッドを確認する。

```sh
kubectl get pods demo-dp -o wide
```

```
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
demo-dp   1/1     Running   0          53s   172.17.255.160   worker01.home.local   <none>           <none>
```

2 つのデバイスが割り当てられていることを確認する。

```sh
kubectl exec demo-dp -- env | grep SAMPLE
```

```
SAMPLE_DEVICE2=1
SAMPLE_DEVICE3=1
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

```
pod/demo-dp-overflow created
```

ポッドを確認する。

```sh
kubectl get pods demo-dp-overflow -o wide
```

```
NAME               READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
demo-dp-overflow   0/1     Pending   0          22s   <none>   <none>   <none>           <none>
```

`Pending` のままリソース待ちになる。

## デバイス・プラグインが起動していない場合

デバイス・プラグインを停止してコンテナを停止する。

```sh
crictl stop 03ef3ee8f
```

ポッドを確認する。

```sh
kubectl get pods
```

```
NAME                          READY   STATUS             RESTARTS   AGE
demo-dp                       0/1     CrashLoopBackOff   0          26m
```

デバイス・プラグインを起動して待つ。

```
NAME                          READY   STATUS    RESTARTS        AGE
demo-dp                       1/1     Running   1 (3m32s ago)   28m
```

## 参考
- [#3573 Add Device Manager to kubelet](https://github.com/kubernetes/enhancements/issues/3573)
- [Device Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)
