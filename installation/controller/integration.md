# インテグレーション

CI ツールとして Tekton を構築する。

## インストール

Tekton Pipelines を構築する。

```sh
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

```text
namespace/tekton-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access created
clusterrole.rbac.authorization.k8s.io/tekton-events-controller-cluster-access created
role.rbac.authorization.k8s.io/tekton-pipelines-controller created
role.rbac.authorization.k8s.io/tekton-pipelines-webhook created
role.rbac.authorization.k8s.io/tekton-pipelines-events-controller created
role.rbac.authorization.k8s.io/tekton-pipelines-leader-election created
role.rbac.authorization.k8s.io/tekton-pipelines-info created
serviceaccount/tekton-pipelines-controller created
serviceaccount/tekton-pipelines-webhook created
serviceaccount/tekton-events-controller created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access created
clusterrolebinding.rbac.authorization.k8s.io/tekton-events-controller-cluster-access created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-leaderelection created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-leaderelection created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-info created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-events-controller created
rolebinding.rbac.authorization.k8s.io/tekton-events-controller-leaderelection created
customresourcedefinition.apiextensions.k8s.io/customruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelines.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/resolutionrequests.resolution.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/stepactions.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/tasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/taskruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/verificationpolicies.tekton.dev created
secret/webhook-certs created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.pipeline.tekton.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.pipeline.tekton.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.pipeline.tekton.dev created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-edit created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-view created
configmap/config-defaults created
configmap/config-events created
configmap/feature-flags created
configmap/pipelines-info created
configmap/config-leader-election-controller created
configmap/config-leader-election-events created
configmap/config-leader-election-webhook created
configmap/config-logging created
configmap/config-observability created
configmap/config-registry-cert created
configmap/config-spire created
configmap/config-tracing created
configmap/config-wait-exponential-backoff created
deployment.apps/tekton-pipelines-controller created
service/tekton-pipelines-controller created
deployment.apps/tekton-events-controller created
service/tekton-events-controller created
namespace/tekton-pipelines-resolvers created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-resolvers-resolution-request-updates created
role.rbac.authorization.k8s.io/tekton-pipelines-resolvers-namespace-rbac created
serviceaccount/tekton-pipelines-resolvers created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-resolvers created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-resolvers-namespace-rbac created
configmap/bundleresolver-config created
configmap/cluster-resolver-config created
configmap/resolvers-feature-flags created
configmap/config-leader-election-resolvers created
configmap/config-logging created
configmap/config-observability created
configmap/git-resolver-config created
configmap/http-resolver-config created
configmap/hubresolver-config created
configmap/resolver-cache-config created
deployment.apps/tekton-pipelines-remote-resolvers created
service/tekton-pipelines-remote-resolvers created
horizontalpodautoscaler.autoscaling/tekton-pipelines-webhook created
deployment.apps/tekton-pipelines-webhook created
service/tekton-pipelines-webhook created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n tekton-pipelines get pod -o wide
```

```text
NAME                                           READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
tekton-events-controller-bcd5b75f7-hqsdg       1/1     Running   0          83s   172.17.51.179    worker02.home.local   <none>           <none>
tekton-pipelines-controller-868956fb6c-9ddfq   1/1     Running   0          83s   172.17.51.181    worker02.home.local   <none>           <none>
tekton-pipelines-webhook-b974bd4b-qvf66        1/1     Running   0          83s   172.17.255.129   worker01.home.local   <none>           <none>
```

クラスタにあるリソースを確認する。

```sh
kubectl -n tekton-pipelines get all
```

```text
NAME                                               READY   STATUS    RESTARTS   AGE
pod/tekton-events-controller-bcd5b75f7-hqsdg       1/1     Running   0          110s
pod/tekton-pipelines-controller-868956fb6c-9ddfq   1/1     Running   0          110s
pod/tekton-pipelines-webhook-b974bd4b-qvf66        1/1     Running   0          110s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                              AGE
service/tekton-events-controller      ClusterIP   10.109.42.149    <none>        9090/TCP,8008/TCP,8080/TCP           110s
service/tekton-pipelines-controller   ClusterIP   10.102.152.142   <none>        9090/TCP,8008/TCP,8080/TCP           110s
service/tekton-pipelines-webhook      ClusterIP   10.105.188.231   <none>        9090/TCP,8008/TCP,443/TCP,8080/TCP   110s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tekton-events-controller      1/1     1            1           110s
deployment.apps/tekton-pipelines-controller   1/1     1            1           110s
deployment.apps/tekton-pipelines-webhook      1/1     1            1           110s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/tekton-events-controller-bcd5b75f7       1         1         1       110s
replicaset.apps/tekton-pipelines-controller-868956fb6c   1         1         1       110s
replicaset.apps/tekton-pipelines-webhook-b974bd4b        1         1         1       110s

NAME                                                           REFERENCE                             TARGETS               MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/tekton-pipelines-webhook   Deployment/tekton-pipelines-webhook   cpu: <unknown>/100%   1         5         1          110s
```

Tekton Triggers を構築する。

```sh
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
```

```text
clusterrole.rbac.authorization.k8s.io/tekton-triggers-admin created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-core-interceptors created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-core-interceptors-secrets created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-eventlistener-roles created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-eventlistener-clusterroles created
role.rbac.authorization.k8s.io/tekton-triggers-admin-webhook created
role.rbac.authorization.k8s.io/tekton-triggers-core-interceptors created
role.rbac.authorization.k8s.io/tekton-triggers-info created
serviceaccount/tekton-triggers-controller created
serviceaccount/tekton-triggers-webhook created
serviceaccount/tekton-triggers-core-interceptors created
clusterrolebinding.rbac.authorization.k8s.io/tekton-triggers-controller-admin created
clusterrolebinding.rbac.authorization.k8s.io/tekton-triggers-webhook-admin created
clusterrolebinding.rbac.authorization.k8s.io/tekton-triggers-core-interceptors created
clusterrolebinding.rbac.authorization.k8s.io/tekton-triggers-core-interceptors-secrets created
rolebinding.rbac.authorization.k8s.io/tekton-triggers-webhook-admin created
rolebinding.rbac.authorization.k8s.io/tekton-triggers-core-interceptors created
rolebinding.rbac.authorization.k8s.io/tekton-triggers-info created
customresourcedefinition.apiextensions.k8s.io/clusterinterceptors.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/clustertriggerbindings.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/eventlisteners.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/interceptors.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/triggers.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/triggerbindings.triggers.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/triggertemplates.triggers.tekton.dev created
secret/triggers-webhook-certs created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.triggers.tekton.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.triggers.tekton.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.triggers.tekton.dev created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-aggregate-edit created
clusterrole.rbac.authorization.k8s.io/tekton-triggers-aggregate-view created
configmap/config-defaults-triggers created
configmap/feature-flags-triggers created
configmap/triggers-info created
configmap/config-leader-election-triggers-controller created
configmap/config-leader-election-triggers-webhook created
configmap/config-logging-triggers created
configmap/config-observability-triggers created
service/tekton-triggers-controller created
deployment.apps/tekton-triggers-controller created
service/tekton-triggers-webhook created
deployment.apps/tekton-triggers-webhook created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n tekton-pipelines get pod -o wide
```

```text
NAME                                           READY   STATUS    RESTARTS   AGE     IP               NODE                  NOMINATED NODE   READINESS GATES
tekton-events-controller-bcd5b75f7-hqsdg       1/1     Running   0          6m18s   172.17.51.179    worker02.home.local   <none>           <none>
tekton-pipelines-controller-868956fb6c-9ddfq   1/1     Running   0          6m18s   172.17.51.181    worker02.home.local   <none>           <none>
tekton-pipelines-webhook-b974bd4b-qvf66        1/1     Running   0          6m18s   172.17.255.129   worker01.home.local   <none>           <none>
tekton-triggers-controller-5bf8589659-gr9jh    1/1     Running   0          51s     172.17.255.179   worker01.home.local   <none>           <none>
tekton-triggers-webhook-557d444bd6-6rtbc       1/1     Running   0          51s     172.17.255.181   worker01.home.local   <none>           <none>
```

オプションの Interceptor を構築する。

```sh
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

```text
secret/tekton-triggers-core-interceptors-certs created
deployment.apps/tekton-triggers-core-interceptors created
service/tekton-triggers-core-interceptors created
clusterinterceptor.triggers.tekton.dev/cel created
clusterinterceptor.triggers.tekton.dev/bitbucket created
clusterinterceptor.triggers.tekton.dev/slack created
clusterinterceptor.triggers.tekton.dev/github created
clusterinterceptor.triggers.tekton.dev/gitlab created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n tekton-pipelines get pod -o wide
```

```text
NAME                                                READY   STATUS    RESTARTS   AGE     IP               NODE                  NOMINATED NODE   READINESS GATES
tekton-events-controller-bcd5b75f7-hqsdg            1/1     Running   0          9m5s    172.17.51.179    worker02.home.local   <none>           <none>
tekton-pipelines-controller-868956fb6c-9ddfq        1/1     Running   0          9m5s    172.17.51.181    worker02.home.local   <none>           <none>
tekton-pipelines-webhook-b974bd4b-qvf66             1/1     Running   0          9m5s    172.17.255.129   worker01.home.local   <none>           <none>
tekton-triggers-controller-5bf8589659-gr9jh         1/1     Running   0          3m38s   172.17.255.179   worker01.home.local   <none>           <none>
tekton-triggers-core-interceptors-d6cc96c67-gjh2v   1/1     Running   0          2m10s   172.17.51.170    worker02.home.local   <none>           <none>
tekton-triggers-webhook-557d444bd6-6rtbc            1/1     Running   0          3m38s   172.17.255.181   worker01.home.local   <none>           <none>
```

クラスタにあるリソースを確認する。

```sh
kubectl -n tekton-pipelines get all
```

```text
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/tekton-events-controller-bcd5b75f7-hqsdg            1/1     Running   0          9m49s
pod/tekton-pipelines-controller-868956fb6c-9ddfq        1/1     Running   0          9m49s
pod/tekton-pipelines-webhook-b974bd4b-qvf66             1/1     Running   0          9m49s
pod/tekton-triggers-controller-5bf8589659-gr9jh         1/1     Running   0          4m22s
pod/tekton-triggers-core-interceptors-d6cc96c67-gjh2v   1/1     Running   0          2m54s
pod/tekton-triggers-webhook-557d444bd6-6rtbc            1/1     Running   0          4m22s

NAME                                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                              AGE
service/tekton-events-controller            ClusterIP   10.109.42.149    <none>        9090/TCP,8008/TCP,8080/TCP           9m49s
service/tekton-pipelines-controller         ClusterIP   10.102.152.142   <none>        9090/TCP,8008/TCP,8080/TCP           9m49s
service/tekton-pipelines-webhook            ClusterIP   10.105.188.231   <none>        9090/TCP,8008/TCP,443/TCP,8080/TCP   9m49s
service/tekton-triggers-controller          ClusterIP   10.111.21.195    <none>        9000/TCP                             4m22s
service/tekton-triggers-core-interceptors   ClusterIP   10.110.127.203   <none>        8443/TCP                             2m54s
service/tekton-triggers-webhook             ClusterIP   10.109.10.173    <none>        443/TCP                              4m22s

NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tekton-events-controller            1/1     1            1           9m49s
deployment.apps/tekton-pipelines-controller         1/1     1            1           9m49s
deployment.apps/tekton-pipelines-webhook            1/1     1            1           9m49s
deployment.apps/tekton-triggers-controller          1/1     1            1           4m22s
deployment.apps/tekton-triggers-core-interceptors   1/1     1            1           2m54s
deployment.apps/tekton-triggers-webhook             1/1     1            1           4m22s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/tekton-events-controller-bcd5b75f7            1         1         1       9m49s
replicaset.apps/tekton-pipelines-controller-868956fb6c        1         1         1       9m49s
replicaset.apps/tekton-pipelines-webhook-b974bd4b             1         1         1       9m49s
replicaset.apps/tekton-triggers-controller-5bf8589659         1         1         1       4m22s
replicaset.apps/tekton-triggers-core-interceptors-d6cc96c67   1         1         1       2m54s
replicaset.apps/tekton-triggers-webhook-557d444bd6            1         1         1       4m22s

NAME                                                           REFERENCE                             TARGETS               MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/tekton-pipelines-webhook   Deployment/tekton-pipelines-webhook   cpu: <unknown>/100%   1         5         1          9m49s
```

Tekton CLI をインストールする。

```sh
TKN_VERSION="0.43.0"
curl -fsSL -o - "https://github.com/tektoncd/cli/releases/download/v${TKN_VERSION}/tkn_${TKN_VERSION}_Linux_x86_64.tar.gz" | \
    tar -zxf - -O tkn > $HOME/.local/bin/tkn
chmod +x $HOME/.local/bin/tkn
```

## 動作確認

タスクを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: hello-world
spec:
  steps:
    - name: show
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello World"
EOF
```

```text
task.tekton.dev/hello-world created
```

タスクを実行する。

```sh
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  name: hello-world-run
spec:
  taskRef:
    name: hello-world
EOF
```

```text
taskrun.tekton.dev/hello-world-run created
```

タスクの実行結果を確認する。

```sh
kubectl get taskrun
```

```text
NAME              SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
hello-world-run   True        Succeeded   29s         15s
```

タスク(ポッド)のログを確認する。

```sh
kubectl logs --selector=tekton.dev/taskRun=hello-world-run
```

```text
Defaulted container "step-show" out of: step-show, prepare (init), place-scripts (init)
Hello World
```

パイプラインを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: hello-world
spec:
  tasks:
    - name: show1
      taskRef:
        name: hello-world
    - name: show2
      runAfter:
        - show1
      taskRef:
        name: hello-world
EOF
```

```text
pipeline.tekton.dev/hello-world created
```

パイプラインを実行する。

```sh
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  name: hello-world-run
spec:
  pipelineRef:
    name: hello-world
EOF
```

```text
pipelinerun.tekton.dev/hello-world-run created
```

パイプラインの実行結果を確認する。

```sh
kubectl get pipelinerun
```

```text
NAME              SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
hello-world-run   True        Succeeded   46s         24s
```

ポッドを確認する。

```sh
kubectl get pods --selector=tekton.dev/pipelineRun=hello-world-run
```

```text
NAME                        READY   STATUS      RESTARTS   AGE
hello-world-run-show1-pod   0/1     Completed   0          2m36s
hello-world-run-show2-pod   0/1     Completed   0          2m30s
```

パイプラインのログを確認する。

```sh
tkn pipelinerun logs hello-world-run
```

```text
[show1 : show] Hello World

[show2 : show] Hello World

```

トリガーテンプレートを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: hello-world-template
spec:
  resourcetemplates:
  - apiVersion: tekton.dev/v1
    kind: PipelineRun
    metadata:
      generateName: hello-world-run-
    spec:
      pipelineRef:
        name: hello-world
EOF
```

```text
triggertemplate.triggers.tekton.dev/hello-world-template created
```

イベントリスナーを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-hello-world
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: hello-world-triggers-eventlistener-binding
subjects:
- kind: ServiceAccount
  name: tekton-hello-world
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hello-world-triggers-eventlistener-clusterbinding
subjects:
- kind: ServiceAccount
  name: tekton-hello-world
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
---
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: hello-world-listener
spec:
  serviceAccountName: tekton-hello-world
  triggers:
    - name: hello-world-trigger
      template:
        ref: hello-world-template
EOF
```

```text
serviceaccount/tekton-hello-world created
rolebinding.rbac.authorization.k8s.io/hello-world-triggers-eventlistener-binding created
clusterrolebinding.rbac.authorization.k8s.io/hello-world-triggers-eventlistener-clusterbinding created
eventlistener.triggers.tekton.dev/hello-world-listener created
```

クラスタにあるリソースを確認する。

```sh
kubectl get all --selector=eventlistener=hello-world-listener
```

```text
NAME                                          READY   STATUS    RESTARTS   AGE
pod/el-hello-world-listener-8f4d55679-6gxlx   1/1     Running   0          2m49s

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/el-hello-world-listener   ClusterIP   10.100.113.165   <none>        8080/TCP,9000/TCP   2m49s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/el-hello-world-listener   1/1     1            1           2m49s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/el-hello-world-listener-8f4d55679   1         1         1       2m49s
```

トリガーを実行する。

イベントリスナーを公開する。

```sh
kubectl port-forward service/el-hello-world-listener 8080
```

```text
Forwarding from 127.0.0.1:8080 -> 8080
```

リクエストする。

```sh
curl -v -H 'content-Type: application/json' -d '{}' http://localhost:8080
```

```text
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.76.1
> Accept: */*
> content-Type: application/json
> Content-Length: 2
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 202 Accepted
< Content-Type: application/json
< Date: Mon, 01 Dec 2025 13:17:18 GMT
< Content-Length: 170
<
{"eventListener":"hello-world-listener","namespace":"default","eventListenerUID":"107cb097-fe28-4865-a2ea-b6cb9371d67b","eventID":"e67c0438-442c-409e-bc1a-734a1584e014"}
* Connection #0 to host localhost left intact
```

パイプラインの実行結果を確認する。

```sh
kubectl get pipelineruns
```

```text
NAME                    SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
hello-world-run         True        Succeeded   86m         86m
hello-world-run-jl8zt   True        Succeeded   2m41s       2m28s
```

パイプラインのログを確認する。

```sh
tkn pipelinerun logs --last -f
```

```text
[show1 : show] Hello World

[show2 : show] Hello World

```
