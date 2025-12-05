# ビルド

Tekton を利用してビルドする。

## パイプラインの準備

リポジトリクローン用のトークンを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: forgejo-token
  annotations:
    tekton.dev/git-0: http://192.168.0.34:3000
type: kubernetes.io/basic-auth
stringData:
  username: forgejo
  password: f5305759e6e7db6c45f108bff252365458f6c708
EOF
```

```text
secret/forgejo-token created
```

トリガーテンプレートを作成する。

```sh
kubectl apply -f - <<"EOF"
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: forgejo-repo-template
spec:
  params:
    - name: ref-name
    - name: repository-url
    - name: revision
  resourcetemplates:
    - apiVersion: tekton.dev/v1
      kind: TaskRun
      metadata:
        generateName: forgejo-repo-run-
      spec:
        serviceAccountName: forgejo-repo-build
        taskSpec:
          steps:
            - image: ubuntu
              script: |
                #!/bin/bash
                apt-get update
                apt-get install -y git

                git clone "$(tt.params.repository-url)" ./app

                pushd ./app
                cat README.md
EOF
```

```text
triggertemplate.triggers.tekton.dev/forgejo-repo-template created
```

トリガーバインディングを作成する。

```sh
kubectl apply -f - <<"EOF"
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: forgejo-event-binding
spec:
  params:
    - name: ref-name
      value: $(body.ref)
    - name: repository-url
      value: $(body.repository.clone_url)
    - name: revision
      value: $(body.after)
EOF
```

```text
triggerbinding.triggers.tekton.dev/forgejo-event-binding created
```

イベントリスナーを作成する。

```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: forgejo-repo-build
secrets:
  - name: forgejo-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: forgejo-repo-triggers-eventlistener-binding
subjects:
- kind: ServiceAccount
  name: forgejo-repo-build
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: forgejo-repo-triggers-eventlistener-clusterbinding
subjects:
- kind: ServiceAccount
  name: forgejo-repo-build
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
---
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: forgejo-repo-listener
spec:
  serviceAccountName: forgejo-repo-build
  triggers:
    - name: forgejo-repo-listener
      interceptors:
        - name: push event only
          ref:
            name: cel
          params:
            - name: filter
              value: "header['X-Forgejo-Event'][0] == 'push'"
      bindings:
        - ref: forgejo-event-binding
      template:
        ref: forgejo-repo-template
  resources:
    kubernetesResource:
      serviceType: LoadBalancer
EOF
```

```text
serviceaccount/forgejo-repo-build created
rolebinding.rbac.authorization.k8s.io/forgejo-repo-triggers-eventlistener-binding created
clusterrolebinding.rbac.authorization.k8s.io/forgejo-repo-triggers-eventlistener-clusterbinding created
eventlistener.triggers.tekton.dev/forgejo-repo-listener created
```

クラスタにあるリソースを確認する。

```sh
kubectl get all --selector=eventlistener=forgejo-repo-listener
```

```text
NAME                                            READY   STATUS    RESTARTS      AGE
pod/el-forgejo-repo-listener-64545b49bc-wn69f   1/1     Running   3 (14m ago)   14m

NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                         AGE
service/el-forgejo-repo-listener   LoadBalancer   10.109.120.41   172.16.0.101   8080:30451/TCP,9000:31845/TCP   14m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/el-forgejo-repo-listener   1/1     1            1           14m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/el-forgejo-repo-listener-64545b49bc   1         1         1       14m
```

## 動作確認

Forgejo の Webhook に下記を設定してリポジトリに Push する。

```text
http://172.16.0.101:8080
```

Webhook が実行されると下記がレスポンスとして返却される。

```json
{
    "eventListener":"forgejo-repo-listener",
    "namespace":"default",
    "eventListenerUID":"99484941-5d88-4921-a1bc-8a076f4a025a",
    "eventID":"5d989933-e67e-45ad-aa18-3f92fd4d6d7d"
}
```

タスクの実行結果を確認する。

```sh
kubectl get taskrun --selector=triggers.tekton.dev/eventlistener=forgejo-repo-listener
```

```text
NAME                     SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
forgejo-repo-run-f2dcv   True        Succeeded   4m31s       4m19s
```

タスクのログを確認する。

```sh
tkn taskrun logs forgejo-repo-run-f2dcv
```

```text
 :
 :
[unnamed-0] 0 added, 0 removed; done.
[unnamed-0] Running hooks in /etc/ca-certificates/update.d...
[unnamed-0] done.
[unnamed-0] Cloning into './app'...
[unnamed-0] /app /
[unnamed-0] Hello, Tekton.

```
