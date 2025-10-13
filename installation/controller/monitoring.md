# モニタリング

モニタリングとして Prometheus を構築する。

## インストール

git をインストールする。

```sh
dnf install -y git-core
```

マニフェストをダウンロードする。

```sh
git clone --depth 1 -b release-0.16 "https://github.com/prometheus-operator/kube-prometheus.git"
cd kube-prometheus
```

名前空間とカスタムリソース定義を構築する。

```sh
kubectl create -f manifests/setup
```

```text
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
namespace/monitoring created
```

リソースの作成を確認する。

```sh
kubectl get servicemonitors --all-namespaces
```

```text
No resources found
```

Prometheus / Alertmanager / Grafana を構築する。

```sh
kubectl create -f manifests/
```

```text
alertmanager.monitoring.coreos.com/main created
networkpolicy.networking.k8s.io/alertmanager-main created
poddisruptionbudget.policy/alertmanager-main created
prometheusrule.monitoring.coreos.com/alertmanager-main-rules created
secret/alertmanager-main created
service/alertmanager-main created
serviceaccount/alertmanager-main created
servicemonitor.monitoring.coreos.com/alertmanager-main created
clusterrole.rbac.authorization.k8s.io/blackbox-exporter created
clusterrolebinding.rbac.authorization.k8s.io/blackbox-exporter created
configmap/blackbox-exporter-configuration created
deployment.apps/blackbox-exporter created
networkpolicy.networking.k8s.io/blackbox-exporter created
service/blackbox-exporter created
serviceaccount/blackbox-exporter created
servicemonitor.monitoring.coreos.com/blackbox-exporter created
secret/grafana-config created
secret/grafana-datasources created
configmap/grafana-dashboard-alertmanager-overview created
configmap/grafana-dashboard-apiserver created
configmap/grafana-dashboard-cluster-total created
configmap/grafana-dashboard-controller-manager created
configmap/grafana-dashboard-grafana-overview created
configmap/grafana-dashboard-k8s-resources-cluster created
configmap/grafana-dashboard-k8s-resources-multicluster created
configmap/grafana-dashboard-k8s-resources-namespace created
configmap/grafana-dashboard-k8s-resources-node created
configmap/grafana-dashboard-k8s-resources-pod created
configmap/grafana-dashboard-k8s-resources-windows-cluster created
configmap/grafana-dashboard-k8s-resources-windows-namespace created
configmap/grafana-dashboard-k8s-resources-windows-pod created
configmap/grafana-dashboard-k8s-resources-workload created
configmap/grafana-dashboard-k8s-resources-workloads-namespace created
configmap/grafana-dashboard-k8s-windows-cluster-rsrc-use created
configmap/grafana-dashboard-k8s-windows-node-rsrc-use created
configmap/grafana-dashboard-kubelet created
configmap/grafana-dashboard-namespace-by-pod created
configmap/grafana-dashboard-namespace-by-workload created
configmap/grafana-dashboard-node-cluster-rsrc-use created
configmap/grafana-dashboard-node-rsrc-use created
configmap/grafana-dashboard-nodes-aix created
configmap/grafana-dashboard-nodes-darwin created
configmap/grafana-dashboard-nodes created
configmap/grafana-dashboard-persistentvolumesusage created
configmap/grafana-dashboard-pod-total created
configmap/grafana-dashboard-prometheus-remote-write created
configmap/grafana-dashboard-prometheus created
configmap/grafana-dashboard-proxy created
configmap/grafana-dashboard-scheduler created
configmap/grafana-dashboard-workload-total created
configmap/grafana-dashboards created
deployment.apps/grafana created
networkpolicy.networking.k8s.io/grafana created
prometheusrule.monitoring.coreos.com/grafana-rules created
service/grafana created
serviceaccount/grafana created
servicemonitor.monitoring.coreos.com/grafana created
prometheusrule.monitoring.coreos.com/kube-prometheus-rules created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
deployment.apps/kube-state-metrics created
networkpolicy.networking.k8s.io/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kube-state-metrics-rules created
service/kube-state-metrics created
serviceaccount/kube-state-metrics created
servicemonitor.monitoring.coreos.com/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kubernetes-monitoring-rules created
servicemonitor.monitoring.coreos.com/kube-apiserver created
servicemonitor.monitoring.coreos.com/coredns created
servicemonitor.monitoring.coreos.com/kube-controller-manager created
servicemonitor.monitoring.coreos.com/kube-scheduler created
servicemonitor.monitoring.coreos.com/kubelet created
clusterrole.rbac.authorization.k8s.io/node-exporter created
clusterrolebinding.rbac.authorization.k8s.io/node-exporter created
daemonset.apps/node-exporter created
networkpolicy.networking.k8s.io/node-exporter created
prometheusrule.monitoring.coreos.com/node-exporter-rules created
service/node-exporter created
serviceaccount/node-exporter created
servicemonitor.monitoring.coreos.com/node-exporter created
clusterrole.rbac.authorization.k8s.io/prometheus-k8s created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-k8s created
networkpolicy.networking.k8s.io/prometheus-k8s created
poddisruptionbudget.policy/prometheus-k8s created
prometheus.monitoring.coreos.com/k8s created
prometheusrule.monitoring.coreos.com/prometheus-k8s-prometheus-rules created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s-config created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s-config created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
service/prometheus-k8s created
serviceaccount/prometheus-k8s created
servicemonitor.monitoring.coreos.com/prometheus-k8s created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrole.rbac.authorization.k8s.io/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter created
clusterrolebinding.rbac.authorization.k8s.io/resource-metrics:system:auth-delegator created
clusterrole.rbac.authorization.k8s.io/resource-metrics-server-resources created
configmap/adapter-config created
deployment.apps/prometheus-adapter created
networkpolicy.networking.k8s.io/prometheus-adapter created
poddisruptionbudget.policy/prometheus-adapter created
rolebinding.rbac.authorization.k8s.io/resource-metrics-auth-reader created
service/prometheus-adapter created
serviceaccount/prometheus-adapter created
servicemonitor.monitoring.coreos.com/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
networkpolicy.networking.k8s.io/prometheus-operator created
prometheusrule.monitoring.coreos.com/prometheus-operator-rules created
service/prometheus-operator created
serviceaccount/prometheus-operator created
servicemonitor.monitoring.coreos.com/prometheus-operator created
```

リソースを確認する。

```sh
kubectl get all -n monitoring -o wide
```

```text
NAME                                       READY   STATUS    RESTARTS   AGE    IP            NODE                    NOMINATED NODE   READINESS GATES
pod/alertmanager-main-0                    2/2     Running   0          70s    172.17.2.1    controller.home.local   <none>           <none>
pod/alertmanager-main-1                    2/2     Running   0          70s    172.17.2.3    controller.home.local   <none>           <none>
pod/alertmanager-main-2                    2/2     Running   0          70s    172.17.2.5    controller.home.local   <none>           <none>
pod/blackbox-exporter-6d86f57b57-ql8s7     3/3     Running   0          118s   172.17.2.4    controller.home.local   <none>           <none>
pod/grafana-7c68d76c67-sbskf               1/1     Running   0          116s   172.17.2.9    controller.home.local   <none>           <none>
pod/kube-state-metrics-c66bdcf9c-p8ws7     3/3     Running   0          116s   172.17.2.6    controller.home.local   <none>           <none>
pod/node-exporter-jsf6t                    2/2     Running   0          115s   172.16.0.11   controller.home.local   <none>           <none>
pod/prometheus-adapter-599c88b6c4-r8944    1/1     Running   0          115s   172.17.2.2    controller.home.local   <none>           <none>
pod/prometheus-adapter-599c88b6c4-tmkmh    1/1     Running   0          115s   172.17.2.8    controller.home.local   <none>           <none>
pod/prometheus-k8s-0                       2/2     Running   0          70s    172.17.2.21   controller.home.local   <none>           <none>
pod/prometheus-k8s-1                       2/2     Running   0          70s    172.17.2.18   controller.home.local   <none>           <none>
pod/prometheus-operator-6fccbfd7fb-qgj5s   2/2     Running   0          114s   172.17.2.7    controller.home.local   <none>           <none>

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE    SELECTOR
service/alertmanager-main       ClusterIP   10.101.224.202   <none>        9093/TCP,8080/TCP            118s   app.kubernetes.io/component=alert-router,app.kubernetes.io/instance=main,app.kubernetes.io/name=alertmanager,app.kubernetes.io/part-of=kube-prometheus
service/alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   70s    app.kubernetes.io/name=alertmanager
service/blackbox-exporter       ClusterIP   10.108.205.136   <none>        9115/TCP,19115/TCP           118s   app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus
service/grafana                 ClusterIP   10.104.116.112   <none>        3000/TCP                     116s   app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus
service/kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP            116s   app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus
service/node-exporter           ClusterIP   None             <none>        9100/TCP                     115s   app.kubernetes.io/component=exporter,app.kubernetes.io/name=node-exporter,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-adapter      ClusterIP   10.106.16.160    <none>        443/TCP                      115s   app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-k8s          ClusterIP   10.97.92.88      <none>        9090/TCP,8080/TCP            115s   app.kubernetes.io/component=prometheus,app.kubernetes.io/instance=k8s,app.kubernetes.io/name=prometheus,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-operated     ClusterIP   None             <none>        9090/TCP                     70s    app.kubernetes.io/name=prometheus
service/prometheus-operator     ClusterIP   None             <none>        8443/TCP                     114s   app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE    CONTAINERS                      IMAGES                                                                           SELECTOR
daemonset.apps/node-exporter   1         1         1       1            1           kubernetes.io/os=linux   115s   node-exporter,kube-rbac-proxy   quay.io/prometheus/node-exporter:v1.9.1,quay.io/brancz/kube-rbac-proxy:v0.19.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=node-exporter,app.kubernetes.io/part-of=kube-prometheus

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS                                                     IMAGES                                                                                                                                        SELECTOR
deployment.apps/blackbox-exporter     1/1     1            1           118s   blackbox-exporter,module-configmap-reloader,kube-rbac-proxy    quay.io/prometheus/blackbox-exporter:v0.27.0,ghcr.io/jimmidyson/configmap-reload:v0.15.0,quay.io/brancz/kube-rbac-proxy:v0.19.1               app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/grafana               1/1     1            1           116s   grafana                                                        grafana/grafana:12.1.0                                                                                                                        app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/kube-state-metrics    1/1     1            1           116s   kube-state-metrics,kube-rbac-proxy-main,kube-rbac-proxy-self   registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.16.0,quay.io/brancz/kube-rbac-proxy:v0.19.1,quay.io/brancz/kube-rbac-proxy:v0.19.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/prometheus-adapter    2/2     2            2           115s   prometheus-adapter                                             registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.12.0                                                                                 app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/prometheus-operator   1/1     1            1           114s   prometheus-operator,kube-rbac-proxy                            quay.io/prometheus-operator/prometheus-operator:v0.85.0,quay.io/brancz/kube-rbac-proxy:v0.19.1                                                app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus

NAME                                             DESIRED   CURRENT   READY   AGE    CONTAINERS                                                     IMAGES                                                                                                                                        SELECTOR
replicaset.apps/blackbox-exporter-6d86f57b57     1         1         1       118s   blackbox-exporter,module-configmap-reloader,kube-rbac-proxy    quay.io/prometheus/blackbox-exporter:v0.27.0,ghcr.io/jimmidyson/configmap-reload:v0.15.0,quay.io/brancz/kube-rbac-proxy:v0.19.1               app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=6d86f57b57
replicaset.apps/grafana-7c68d76c67               1         1         1       116s   grafana                                                        grafana/grafana:12.1.0                                                                                                                        app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=7c68d76c67
replicaset.apps/kube-state-metrics-c66bdcf9c     1         1         1       116s   kube-state-metrics,kube-rbac-proxy-main,kube-rbac-proxy-self   registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.16.0,quay.io/brancz/kube-rbac-proxy:v0.19.1,quay.io/brancz/kube-rbac-proxy:v0.19.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=c66bdcf9c
replicaset.apps/prometheus-adapter-599c88b6c4    2         2         2       115s   prometheus-adapter                                             registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.12.0                                                                                 app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=599c88b6c4
replicaset.apps/prometheus-operator-6fccbfd7fb   1         1         1       114s   prometheus-operator,kube-rbac-proxy                            quay.io/prometheus-operator/prometheus-operator:v0.85.0,quay.io/brancz/kube-rbac-proxy:v0.19.1                                                app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=6fccbfd7fb

NAME                                 READY   AGE   CONTAINERS                     IMAGES
statefulset.apps/alertmanager-main   3/3     70s   alertmanager,config-reloader   quay.io/prometheus/alertmanager:v0.28.1,quay.io/prometheus-operator/prometheus-config-reloader:v0.85.0
statefulset.apps/prometheus-k8s      2/2     70s   prometheus,config-reloader     quay.io/prometheus/prometheus:v3.5.0,quay.io/prometheus-operator/prometheus-config-reloader:v0.85.0
```

## 動作確認

Prometheus サービスをプロキシする。

```sh
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090 --address 0.0.0.0
```

Grafana サービスをプロキシする。
admin/admin でログインする。初回にパスワードを更新する。

```sh
kubectl --namespace monitoring port-forward svc/grafana 3000 --address 0.0.0.0
```

Alertmanager サービスをプロキシする。

```sh
kubectl --namespace monitoring port-forward svc/alertmanager-main 9093 --address 0.0.0.0
```

## Ingress

Prometheus サービスのネットワークポリシーに下記を追加して ingress からの接続を許可する。

```sh
kubectl -n monitoring edit networkpolicy prometheus-k8s
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
```

```text
networkpolicy.networking.k8s.io/prometheus-k8s edited
```

Prometheus サービスの ingress を作成する。

```sh
kubectl -n monitoring create ingress \
    prometheus-k8s \
    --class=nginx \
    --annotation="nginx.ingress.kubernetes.io/ssl-redirect=false" \
    --rule="prometheus/*=prometheus-k8s:9090"
```

```text
ingress.networking.k8s.io/prometheus-k8s created
```

ingress を確認する。

```sh
kubectl -n monitoring get ingress prometheus-k8s
```

```text
NAME             CLASS   HOSTS        ADDRESS        PORTS   AGE
prometheus-k8s   nginx   prometheus   172.16.0.102   80      48s
```

接続確認する。

```sh
curl --resolve prometheus:80:172.16.0.100 http://prometheus/
```

HTML が返却される。

Grafana サービスのネットワークポリシーに下記を追加して ingress からの接続を許可する。

```sh
kubectl -n monitoring edit networkpolicy grafana
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
```

```text
networkpolicy.networking.k8s.io/grafana edited
```

Grafana サービスの ingress を作成する。

```sh
kubectl -n monitoring create ingress \
    grafana \
    --class=nginx \
    --annotation="nginx.ingress.kubernetes.io/ssl-redirect=false" \
    --rule="grafana/*=grafana:3000"
```

```text
ingress.networking.k8s.io/grafana created
```

ingress を確認する。

```sh
kubectl -n monitoring get ingress grafana
```

```text
NAME      CLASS   HOSTS     ADDRESS        PORTS   AGE
grafana   nginx   grafana   172.16.0.102   80      55s
```

接続確認する。

```sh
curl --resolve grafana:80:172.16.0.100 http://grafana/
```

HTML が返却される。

Alertmanager サービスのネットワークポリシーに下記を追加して ingress からの接続を許可する。

```sh
kubectl -n monitoring edit networkpolicy alertmanager-main
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
```

```text
networkpolicy.networking.k8s.io/alertmanager-main edited
```

Alertmanager サービスの ingress を作成する。

```sh
kubectl -n monitoring create ingress \
    alertmanager \
    --class=nginx \
    --annotation="nginx.ingress.kubernetes.io/ssl-redirect=false" \
    --rule="alertmanager/*=alertmanager-main:9093"
```

```text
ingress.networking.k8s.io/alertmanager created
```

ingress を確認する。

```sh
kubectl -n monitoring get ingress alertmanager
```

```text
NAME           CLASS   HOSTS          ADDRESS        PORTS   AGE
alertmanager   nginx   alertmanager   172.16.0.102   80      57s
```

接続確認する。

```sh
curl --resolve alertmanager:80:172.16.0.100 http://alertmanager/
```

HTML が返却される。
