# モニタリング

モニタリングとして Prometheus を構築する。

## インストール

マニフェストをダウンロードする。

```sh
git clone --depth 1 -b release-0.14 https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
```

名前空間とカスタムリソース定義を構築する。

```sh
kubectl apply --server-side -f manifests/setup
```

```text
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com serverside-applied
namespace/monitoring serverside-applied
```

リソースの作成を確認する。

```sh
kubectl wait \
    --for condition=Established \
    --all CustomResourceDefinition \
    --namespace=monitoring
```

```text
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io condition met
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io condition met
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io condition met
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io condition met
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io condition met
customresourcedefinition.apiextensions.k8s.io/ingressclassparameterses.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io condition met
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/kongclusterplugins.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongconsumergroups.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongconsumers.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongingresses.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/konglicenses.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongplugins.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongupstreampolicies.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kongvaults.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org condition met
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/servicel2statuses.metallb.io condition met
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/tcpingresses.configuration.konghq.com condition met
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io condition met
customresourcedefinition.apiextensions.k8s.io/udpingresses.configuration.konghq.com condition met
```

Prometheus / Alertmanager / Grafana を構築する。

```sh
kubectl apply -f manifests/
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
configmap/grafana-dashboard-k8s-resources-workload created
configmap/grafana-dashboard-k8s-resources-workloads-namespace created
configmap/grafana-dashboard-kubelet created
configmap/grafana-dashboard-namespace-by-pod created
configmap/grafana-dashboard-namespace-by-workload created
configmap/grafana-dashboard-node-cluster-rsrc-use created
configmap/grafana-dashboard-node-rsrc-use created
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
kubectl -n monitoring get all -o wide
```

```text
NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE                    NOMINATED NODE   READINESS GATES
pod/alertmanager-main-0                    2/2     Running   0          2m41s   172.17.255.151   worker01.home.local     <none>           <none>
pod/alertmanager-main-1                    2/2     Running   0          2m41s   172.17.2.46      controller.home.local   <none>           <none>
pod/alertmanager-main-2                    2/2     Running   0          2m41s   172.17.51.157    worker02.home.local     <none>           <none>
pod/blackbox-exporter-86745676c9-lcjm8     3/3     Running   0          3m10s   172.17.255.177   worker01.home.local     <none>           <none>
pod/grafana-599bb4cc9d-j5vqr               1/1     Running   0          3m9s    172.17.255.136   worker01.home.local     <none>           <none>
pod/kube-state-metrics-6f46974967-r5fgm    3/3     Running   0          3m9s    172.17.255.180   worker01.home.local     <none>           <none>
pod/node-exporter-srv9s                    2/2     Running   0          3m9s    172.16.0.32      worker02.home.local     <none>           <none>
pod/node-exporter-t8js4                    2/2     Running   0          3m9s    172.16.0.31      worker01.home.local     <none>           <none>
pod/node-exporter-vlzpj                    2/2     Running   0          3m9s    172.16.0.11      controller.home.local   <none>           <none>
pod/prometheus-adapter-784f566c54-s8sck    1/1     Running   0          3m9s    172.17.51.161    worker02.home.local     <none>           <none>
pod/prometheus-adapter-784f566c54-zpjg2    1/1     Running   0          3m9s    172.17.255.191   worker01.home.local     <none>           <none>
pod/prometheus-k8s-0                       2/2     Running   0          2m41s   172.17.255.137   worker01.home.local     <none>           <none>
pod/prometheus-k8s-1                       2/2     Running   0          2m41s   172.17.51.154    worker02.home.local     <none>           <none>
pod/prometheus-operator-57b579d5b9-fsc5g   2/2     Running   0          3m9s    172.17.51.151    worker02.home.local     <none>           <none>

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE     SELECTOR
service/alertmanager-main       ClusterIP   10.102.81.157    <none>        9093/TCP,8080/TCP            3m10s   app.kubernetes.io/component=alert-router,app.kubernetes.io/instance=main,app.kubernetes.io/name=alertmanager,app.kubernetes.io/part-of=kube-prometheus
service/alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   2m42s   app.kubernetes.io/name=alertmanager
service/blackbox-exporter       ClusterIP   10.111.167.252   <none>        9115/TCP,19115/TCP           3m10s   app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus
service/grafana                 ClusterIP   10.104.62.79     <none>        3000/TCP                     3m9s    app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus
service/kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP            3m9s    app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus
service/node-exporter           ClusterIP   None             <none>        9100/TCP                     3m9s    app.kubernetes.io/component=exporter,app.kubernetes.io/name=node-exporter,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-adapter      ClusterIP   10.108.56.239    <none>        443/TCP                      3m9s    app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-k8s          ClusterIP   10.106.77.243    <none>        9090/TCP,8080/TCP            3m9s    app.kubernetes.io/component=prometheus,app.kubernetes.io/instance=k8s,app.kubernetes.io/name=prometheus,app.kubernetes.io/part-of=kube-prometheus
service/prometheus-operated     ClusterIP   None             <none>        9090/TCP                     2m41s   app.kubernetes.io/name=prometheus
service/prometheus-operator     ClusterIP   None             <none>        8443/TCP                     3m9s    app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE    CONTAINERS                      IMAGES                                                                           SELECTOR
daemonset.apps/node-exporter   3         3         3       3            3           kubernetes.io/os=linux   3m9s   node-exporter,kube-rbac-proxy   quay.io/prometheus/node-exporter:v1.8.2,quay.io/brancz/kube-rbac-proxy:v0.18.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=node-exporter,app.kubernetes.io/part-of=kube-prometheus

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                                                     IMAGES                                                                                                                                        SELECTOR
deployment.apps/blackbox-exporter     1/1     1            1           3m10s   blackbox-exporter,module-configmap-reloader,kube-rbac-proxy    quay.io/prometheus/blackbox-exporter:v0.25.0,ghcr.io/jimmidyson/configmap-reload:v0.13.1,quay.io/brancz/kube-rbac-proxy:v0.18.1               app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/grafana               1/1     1            1           3m9s    grafana                                                        grafana/grafana:11.2.0                                                                                                                        app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/kube-state-metrics    1/1     1            1           3m9s    kube-state-metrics,kube-rbac-proxy-main,kube-rbac-proxy-self   registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.13.0,quay.io/brancz/kube-rbac-proxy:v0.18.1,quay.io/brancz/kube-rbac-proxy:v0.18.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/prometheus-adapter    2/2     2            2           3m9s    prometheus-adapter                                             registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.12.0                                                                                 app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus
deployment.apps/prometheus-operator   1/1     1            1           3m9s    prometheus-operator,kube-rbac-proxy                            quay.io/prometheus-operator/prometheus-operator:v0.76.2,quay.io/brancz/kube-rbac-proxy:v0.18.1                                                app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus

NAME                                             DESIRED   CURRENT   READY   AGE     CONTAINERS                                                     IMAGES                                                                                                                                        SELECTOR
replicaset.apps/blackbox-exporter-86745676c9     1         1         1       3m10s   blackbox-exporter,module-configmap-reloader,kube-rbac-proxy    quay.io/prometheus/blackbox-exporter:v0.25.0,ghcr.io/jimmidyson/configmap-reload:v0.13.1,quay.io/brancz/kube-rbac-proxy:v0.18.1               app.kubernetes.io/component=exporter,app.kubernetes.io/name=blackbox-exporter,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=86745676c9
replicaset.apps/grafana-599bb4cc9d               1         1         1       3m9s    grafana                                                        grafana/grafana:11.2.0                                                                                                                        app.kubernetes.io/component=grafana,app.kubernetes.io/name=grafana,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=599bb4cc9d
replicaset.apps/kube-state-metrics-6f46974967    1         1         1       3m9s    kube-state-metrics,kube-rbac-proxy-main,kube-rbac-proxy-self   registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.13.0,quay.io/brancz/kube-rbac-proxy:v0.18.1,quay.io/brancz/kube-rbac-proxy:v0.18.1   app.kubernetes.io/component=exporter,app.kubernetes.io/name=kube-state-metrics,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=6f46974967
replicaset.apps/prometheus-adapter-784f566c54    2         2         2       3m9s    prometheus-adapter                                             registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.12.0                                                                                 app.kubernetes.io/component=metrics-adapter,app.kubernetes.io/name=prometheus-adapter,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=784f566c54
replicaset.apps/prometheus-operator-57b579d5b9   1         1         1       3m9s    prometheus-operator,kube-rbac-proxy                            quay.io/prometheus-operator/prometheus-operator:v0.76.2,quay.io/brancz/kube-rbac-proxy:v0.18.1                                                app.kubernetes.io/component=controller,app.kubernetes.io/name=prometheus-operator,app.kubernetes.io/part-of=kube-prometheus,pod-template-hash=57b579d5b9

NAME                                 READY   AGE     CONTAINERS                     IMAGES
statefulset.apps/alertmanager-main   3/3     2m41s   alertmanager,config-reloader   quay.io/prometheus/alertmanager:v0.27.0,quay.io/prometheus-operator/prometheus-config-reloader:v0.76.2
statefulset.apps/prometheus-k8s      2/2     2m41s   prometheus,config-reloader     quay.io/prometheus/prometheus:v2.54.1,quay.io/prometheus-operator/prometheus-config-reloader:v0.76.2
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

Prometheus サービスのネットワークポリシーを更新する。

```sh
kubectl -n monitoring edit networkpolicy prometheus-k8s
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
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
prometheus-k8s   nginx   prometheus   172.16.0.100   80      35s
```

接続確認する。

```sh
curl --resolve prometheus:80:172.16.0.100 http://prometheus/
```

HTML が返却される。

Grafana サービスのネットワークポリシーを更新する。

```sh
kubectl -n monitoring edit networkpolicy grafana
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
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
grafana   nginx   grafana   172.16.0.100   80      33s
```

接続確認する。

```sh
curl --resolve grafana:80:172.16.0.100 http://grafana/
```

HTML が返却される。

Alertmanager サービスのネットワークポリシーを更新する。

```sh
kubectl -n monitoring edit networkpolicy alertmanager-main
```

```yaml
  - from:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
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
alertmanager   nginx   alertmanager   172.16.0.100   80      20s
```

接続確認する。

```sh
curl --resolve alertmanager:80:172.16.0.100 http://alertmanager/
```

HTML が返却される。
