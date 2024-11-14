# ネットワークポリシー

ポッド間の通信を制御する。

ポッドに適用されるポリシーが 1 つでもあると通信の既定の動作は拒否となる。
複数のポリシーが一致する場合はルールの和が適用される。ルールの優先順位はない。

## ポッドの用意

ラベルの異なるポッドを作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-a
  namespace: default
spec:
  selector:
    matchLabels:
      app: policy-a
  template:
    metadata:
      labels:
        app: policy-a
    spec:
      containers:
      - name: pod-a
        image: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-b
  namespace: default
spec:
  selector:
    matchLabels:
      app: policy-b
  template:
    metadata:
      labels:
        app: policy-b
    spec:
      containers:
      - name: pod-b
        image: nginx
        ports:
        - containerPort: 80
          protocol: TCP
EOF
```

ポッドを確認する。

```sh
kubectl get pods -o wide
```

```
NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
pod-a-6745f849d7-8ptnr   1/1     Running   0          69s   172.17.51.169    worker02.home.local   <none>           <none>
pod-b-75f6c585bc-7nw42   1/1     Running   0          69s   172.17.255.152   worker01.home.local   <none>           <none>
```

## ネットワークポリシーの設定

ポッド間の疎通を確認する。

```sh
kubectl exec -t pod-b-75f6c585bc-7nw42 -- curl -s "http://$(kubectl get pod pod-a-6745f849d7-8ptnr -o jsonpath='{.status.podIP}')"
```

HTML が返却される。

```sh
kubectl exec -t pod-a-6745f849d7-8ptnr -- curl -s "http://$(kubectl get pod pod-b-75f6c585bc-7nw42 -o jsonpath='{.status.podIP}')"
```

HTML が返却される。

### 通信をすべて拒否(Ingress)

すべてのポッド間の通信を拒否するネットワークポリシーを作成する(Ingress のみ)。

すべてのポッドに一致するため、すべての通信の動作が既定で拒否となる。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
EOF
```

```
networkpolicy.networking.k8s.io/deny created
```

ネットワークポリシーを確認する。

```sh
kubectl get networkpolicy default-deny
```

```
NAME           POD-SELECTOR   AGE
default-deny   <none>         22m
```

ポッド間の疎通を確認する。

```sh
kubectl exec -t pod-b-75f6c585bc-7nw42 -- curl -s "http://$(kubectl get pod pod-a-6745f849d7-8ptnr -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

```sh
kubectl exec -t pod-a-6745f849d7-8ptnr -- curl -s "http://$(kubectl get pod pod-b-75f6c585bc-7nw42 -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

### 部分的に許可(Ingress)

`policy-a` から　`policy-b` に通信を許可する。

ポリシーはステートフルになる。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ab
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: policy-b
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: policy-a
    ports:
    - port: 80
      protocol: TCP
EOF
```

```
networkpolicy.networking.k8s.io/allow-ab created
```

ネットワークポリシーを確認する。

```sh
kubectl get networkpolicy allow-ab
```

```
NAME       POD-SELECTOR   AGE
allow-ab   app=policy-b   5m31s
```

ポッド間の疎通を確認する。

```sh
kubectl exec -t pod-b-75f6c585bc-7nw42 -- curl -s "http://$(kubectl get pod pod-a-6745f849d7-8ptnr -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

```sh
kubectl exec -t pod-a-6745f849d7-8ptnr -- curl -s "http://$(kubectl get pod pod-b-75f6c585bc-7nw42 -o jsonpath='{.status.podIP}')"
```

HTML が返却される。

### 通信をすべて拒否(Egress)

すべてのポッド間の通信を拒否するネットワークポリシーを作成する(Ingress と Egress)。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
```

```
networkpolicy.networking.k8s.io/default-deny configured
```

ネットワークポリシーを確認する。

```sh
kubectl get networkpolicy default-deny
```

```
NAME           POD-SELECTOR   AGE
default-deny   <none>         21m
```

ポッド間の疎通を確認する。

```sh
kubectl exec -t pod-b-75f6c585bc-7nw42 -- curl -s "http://$(kubectl get pod pod-a-6745f849d7-8ptnr -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

```sh
kubectl exec -t pod-a-6745f849d7-8ptnr -- curl -s "http://$(kubectl get pod pod-b-75f6c585bc-7nw42 -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

### 部分的に許可(Egress)

`policy-a` から　`policy-b` に通信を許可する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ab
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: policy-b
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: policy-a
    ports:
    - port: 80
      protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ba
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: policy-a
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: policy-b
    ports:
    - port: 80
      protocol: TCP
EOF
```

```
networkpolicy.networking.k8s.io/allow-ab unchanged
networkpolicy.networking.k8s.io/allow-ba created
```

ネットワークポリシーを確認する。

```sh
kubectl get networkpolicy allow-ab allow-ba
```

```
NAME       POD-SELECTOR   AGE
allow-ab   app=policy-b   13m
allow-ba   app=policy-a   49s
```

ポッド間の疎通を確認する。

```sh
kubectl exec -t pod-b-75f6c585bc-7nw42 -- curl -s "http://$(kubectl get pod pod-a-6745f849d7-8ptnr -o jsonpath='{.status.podIP}')"
```

制御が返ってこない。

```sh
kubectl exec -t pod-a-6745f849d7-8ptnr -- curl -s "http://$(kubectl get pod pod-b-75f6c585bc-7nw42 -o jsonpath='{.status.podIP}')"
```

HTML が返却される。
