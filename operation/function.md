# ファンクション

Knative でサービスをデプロイする。

## Knative Serving

### 作成

サービスをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
        - image: ghcr.io/knative/helloworld-go:latest
          ports:
            - containerPort: 8080
          env:
            - name: TARGET
              value: "World"
EOF
```

```text
Warning: Kubernetes default value is insecure, Knative may default this to secure in a future release: spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation, spec.template.spec.containers[0].securityContext.capabilities, spec.template.spec.containers[0].securityContext.runAsNonRoot, spec.template.spec.containers[0].securityContext.seccompProfile
service.serving.knative.dev/hello created
```

Kubernetes のリソースを確認する。

```sh
kubectl get all
```

```text
NAME                                          READY   STATUS        RESTARTS   AGE
pod/hello-00001-deployment-5884d678cf-mscfj   2/2     Terminating   0          67s

NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP                                         PORT(S)                                              AGE
service/hello                 ExternalName   <none>          kourier-internal.kourier-system.svc.cluster.local   80/TCP                                               63s
service/hello-00001           ClusterIP      10.105.28.142   <none>                                              80/TCP,443/TCP                                       67s
service/hello-00001-private   ClusterIP      10.100.98.236   <none>                                              80/TCP,443/TCP,9090/TCP,9091/TCP,8022/TCP,8012/TCP   67s
service/kubernetes            ClusterIP      10.96.0.1       <none>                                              443/TCP                                              8d

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-00001-deployment   0/0     0            0           67s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-00001-deployment-5884d678cf   0         0         0       67s

NAME                                      LATESTCREATED   LATESTREADY   READY   REASON
configuration.serving.knative.dev/hello   hello-00001     hello-00001   True

NAME                                       CONFIG NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
revision.serving.knative.dev/hello-00001   hello         1            True             0                 0

NAME                              URL                                       READY   REASON
route.serving.knative.dev/hello   http://hello.default.knative.home.local   True

NAME                                URL                                       LATESTCREATED   LATESTREADY   READY   REASON
service.serving.knative.dev/hello   http://hello.default.knative.home.local   hello-00001     hello-00001   True
```

Knative のサービスを確認する。

```sh
kn service list
```

```text
NAME    URL                                       LATEST        AGE     CONDITIONS   READY   REASON
hello   http://hello.default.knative.home.local   hello-00001   2m59s   3 OK / 3     True
```

サービスを実行する。

```sh
curl http://hello.default.knative.home.local
```

```text
Hello World!
```

### 更新

サービスを更新する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
        - image: ghcr.io/knative/helloworld-go:latest
          ports:
            - containerPort: 8080
          env:
            - name: TARGET
              value: "Knative"
EOF
```

```text
Warning: Kubernetes default value is insecure, Knative may default this to secure in a future release: spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation, spec.template.spec.containers[0].securityContext.capabilities, spec.template.spec.containers[0].securityContext.runAsNonRoot, spec.template.spec.containers[0].securityContext.seccompProfile
service.serving.knative.dev/hello configured
```

サービスを実行する。

```sh
curl http://hello.default.knative.home.local
```

```text
Hello Knative!
```

更新を確認する。

```sh
kn revisions list
```

```text
NAME          SERVICE   TRAFFIC   TAGS   GENERATION   AGE    CONDITIONS   READY   REASON
hello-00002   hello     100%             2            106s   3 OK / 4     True
hello-00001   hello                      1            24m    3 OK / 4     True
```

```sh
kubectl get revisions
```

```text
NAME          CONFIG NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
hello-00001   hello         1            True             0                 0
hello-00002   hello         2            True             1                 1
```

重みを変更する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
        - image: ghcr.io/knative/helloworld-go:latest
          ports:
            - containerPort: 8080
          env:
            - name: TARGET
              value: "Knative"
  traffic:
  - latestRevision: true
    percent: 50
  - latestRevision: false
    percent: 50
    revisionName: hello-00001
EOF
```

```text
Warning: Kubernetes default value is insecure, Knative may default this to secure in a future release: spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation, spec.template.spec.containers[0].securityContext.capabilities, spec.template.spec.containers[0].securityContext.runAsNonRoot, spec.template.spec.containers[0].securityContext.seccompProfile
service.serving.knative.dev/hello configured
```

更新を確認する。

```sh
kn revisions list
```

```text
NAME          SERVICE   TRAFFIC   TAGS   GENERATION   AGE     CONDITIONS   READY   REASON
hello-00002   hello     50%              2            5m12s   3 OK / 4     True
hello-00001   hello     50%              1            27m     3 OK / 4     True
```

サービスを実行する。

```sh
yes http://hello.default.knative.home.local | head -n 10 | xargs curl --parallel
```

```text
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello Knative!
Hello Knative!
Hello Knative!
Hello Knative!
```

## Knative Eventing

### ブローカ

ブローカを作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: dev-broker
EOF
```

```text
broker.eventing.knative.dev/dev-broker created
```

ブローカを確認する。

```sh
kn broker list
```

```text
NAME         URL                                                                           AGE   CONDITIONS   READY   REASON
dev-broker   http://broker-ingress.knative-eventing.svc.cluster.local/default/dev-broker   63s   7 OK / 7     True
```

### ソース

ソースとして CloudEvents Player を作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: cloudevents-player
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/min-scale: "1"
    spec:
      containers:
        - image: quay.io/ruben/cloudevents-player:latest
EOF
```

```text
Warning: Kubernetes default value is insecure, Knative may default this to secure in a future release: spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation, spec.template.spec.containers[0].securityContext.capabilities, spec.template.spec.containers[0].securityContext.runAsNonRoot, spec.template.spec.containers[0].securityContext.seccompProfile
service.serving.knative.dev/cloudevents-player created
```

ポッドが起動するまで待つ。

```sh
watch kubectl get pod -o wide
```

```text
NAME                                                   READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
cloudevents-player-00001-deployment-5784d5b774-mvvqd   2/2     Running   0          51s   172.17.255.142   worker01.home.local   <none>           <none>
```

Knative サービスを確認する。

```sh
kn service list
```

```text
NAME                 URL                                                    LATEST                     AGE    CONDITIONS   READY   REASON
cloudevents-player   http://cloudevents-player.default.knative.home.local   cloudevents-player-00001   2m6s   3 OK / 3     True
```

ブローカと接続する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: ce-player-binding
spec:
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: dev-broker
  subject:
    apiVersion: serving.knative.dev/v1
    kind: Service
    name: cloudevents-player
EOF
```

```text
sinkbinding.sources.knative.dev/ce-player-binding created
```

バインディングを確認する。

```sh
kubectl get sinkbinding
```

```text
NAME                SINK                                                                          AGE   READY   REASON
ce-player-binding   http://broker-ingress.knative-eventing.svc.cluster.local/default/dev-broker   32s   True
```

### シンク

シンクに CloudEvents Player を指定してトリガーを作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: cloudevents-trigger
  annotations:
    knative-eventing-injection: enabled
spec:
  broker: dev-broker
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: cloudevents-player
EOF
```

```text
trigger.eventing.knative.dev/cloudevents-trigger created
```

トリガーを確認する。

```sh
kubectl get trigger
```

```text
NAME                  BROKER       SUBSCRIBER_URI                                        AGE   READY   REASON
cloudevents-trigger   dev-broker   http://cloudevents-player.default.svc.cluster.local   35s   True
```
