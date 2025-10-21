# サーバレス

サーバレス基盤として Knative を構築する。

## インストール

### クライアント

Knative CLI をインストールする。

```sh
KN_VERSION=1.19.5
mkdir -p $HOME/.local/bin
curl -fSL "https://github.com/knative/client/releases/download/knative-v${KN_VERSION}/kn-linux-amd64" \
    --output-dir $HOME/.local/bin \
    -o kn
chmod +x $HOME/.local/bin/kn
```

Knative Function CLI をインストールする。

```sh
KNF_VERSION=1.19.4
mkdir -p $HOME/.local/bin
curl -fSL "https://github.com/knative/func/releases/download/knative-v${KNF_VERSION}/func_linux_amd64" \
    --output-dir $HOME/.local/bin \
    -o func
chmod +x $HOME/.local/bin/func
cp $HOME/.local/bin/func $HOME/.local/bin/kn-func
```

(オプション) Kubernetes CLI をインストールする。

```sh
KUBE_VERSION=$(curl -L -s https://dl.k8s.io/release/stable.txt)
curl -fSLO "https://dl.k8s.io/release/${KUBE_VERSION}/bin/linux/amd64/kubectl" \
    --output-dir $HOME/.local/bin
```

### Knative Serving

Knative Serving のカスタムリソース定義を構築する。

```sh
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.19.6/serving-crds.yaml
```

```text
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev created
```

Knative serving を構築する。

```sh
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.19.6/serving-core.yaml
```

```text
namespace/knative-serving created
role.rbac.authorization.k8s.io/knative-serving-activator created
clusterrole.rbac.authorization.k8s.io/knative-serving-activator-cluster created
clusterrole.rbac.authorization.k8s.io/knative-serving-aggregated-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-edit created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-view created
clusterrole.rbac.authorization.k8s.io/knative-serving-core created
clusterrole.rbac.authorization.k8s.io/knative-serving-podspecable-binding created
serviceaccount/controller created
clusterrole.rbac.authorization.k8s.io/knative-serving-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-addressable-resolver created
serviceaccount/activator created
rolebinding.rbac.authorization.k8s.io/knative-serving-activator created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-activator-cluster created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev unchanged
certificate.networking.internal.knative.dev/routing-serving-certs created
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev unchanged
image.caching.internal.knative.dev/queue-proxy created
configmap/config-autoscaler created
configmap/config-certmanager created
configmap/config-defaults created
configmap/config-deployment created
configmap/config-domain created
configmap/config-features created
configmap/config-gc created
configmap/config-leader-election created
configmap/config-logging created
configmap/config-network created
configmap/config-observability created
configmap/config-tracing created
horizontalpodautoscaler.autoscaling/activator created
poddisruptionbudget.policy/activator-pdb created
deployment.apps/activator created
service/activator-service created
deployment.apps/autoscaler created
service/autoscaler created
deployment.apps/controller created
service/controller created
horizontalpodautoscaler.autoscaling/webhook created
poddisruptionbudget.policy/webhook-pdb created
deployment.apps/webhook created
service/webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.serving.knative.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.serving.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.serving.knative.dev created
secret/webhook-certs created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n knative-serving get pod -o wide
```

```text
NAME                                      READY   STATUS    RESTARTS        AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
activator-6ddbf68b54-7hwpv                1/1     Running   9 (2m40s ago)   34m   172.17.51.148    worker02.home.local   <none>           <none>
autoscaler-5b68c5d94d-27tqt               1/1     Running   0               34m   172.17.255.164   worker01.home.local   <none>           <none>
controller-5659945959-9j2gr               1/1     Running   0               34m   172.17.255.165   worker01.home.local   <none>           <none>
webhook-7d954f9969-pllpw                  1/1     Running   0               34m   172.17.255.166   worker01.home.local   <none>           <none>
```

サービスを確認する。

```sh
kubectl -n knative-serving get service
```

```text
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                   AGE
activator-service            ClusterIP   10.104.163.99   <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   3m53s
autoscaler                   ClusterIP   10.96.106.126   <none>        9090/TCP,8008/TCP,8080/TCP                3m53s
autoscaler-bucket-00-of-01   ClusterIP   10.108.74.110   <none>        8080/TCP                                  3m47s
controller                   ClusterIP   10.109.23.187   <none>        9090/TCP,8008/TCP                         3m53s
webhook                      ClusterIP   10.111.78.35    <none>        9090/TCP,8008/TCP,443/TCP                 3m53s
```

Knative Kourier を構築する。

```sh
kubectl apply -f https://github.com/knative-extensions/net-kourier/releases/download/knative-v1.19.5/kourier.yaml
```

```text
namespace/kourier-system created
configmap/kourier-bootstrap created
configmap/config-kourier created
serviceaccount/net-kourier created
clusterrole.rbac.authorization.k8s.io/net-kourier created
clusterrolebinding.rbac.authorization.k8s.io/net-kourier created
deployment.apps/net-kourier-controller created
service/net-kourier-controller created
deployment.apps/3scale-kourier-gateway created
service/kourier created
service/kourier-internal created
horizontalpodautoscaler.autoscaling/3scale-kourier-gateway created
poddisruptionbudget.policy/3scale-kourier-gateway-pdb created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n kourier-system get pod -o wide
```

```text
NAME                                      READY   STATUS    RESTARTS         AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
3scale-kourier-gateway-78966c4b94-vtxd2   1/1     Running   12 (5m55s ago)   30m   172.17.255.167   worker01.home.local   <none>           <none>
```

サービスを確認する。

```sh
kubectl -n kourier-system get service
```

```text
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
kourier            LoadBalancer   10.100.26.236   172.16.0.103   80:30971/TCP,443:30122/TCP   101s
kourier-internal   ClusterIP      10.103.21.234   <none>         80/TCP,443/TCP               101s
```

Knative Serving のネットワークレイヤーに Knative Kourier を設定する。

```sh
kubectl patch configmap/config-network \
    --namespace knative-serving \
    --type merge \
    --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
```

kourier のポッドが起動する。

```sh
kubectl -n knative-serving get pod -o wide
```

```text
NAME                                      READY   STATUS    RESTARTS        AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
activator-6ddbf68b54-7hwpv                1/1     Running   9 (5m16s ago)   37m   172.17.51.148    worker02.home.local   <none>           <none>
autoscaler-5b68c5d94d-27tqt               1/1     Running   0               37m   172.17.255.164   worker01.home.local   <none>           <none>
controller-5659945959-9j2gr               1/1     Running   0               37m   172.17.255.165   worker01.home.local   <none>           <none>
net-kourier-controller-594bf67869-5fszq   1/1     Running   0               32m   172.17.51.149    worker02.home.local   <none>           <none>
webhook-7d954f9969-pllpw                  1/1     Running   0               37m   172.17.255.166   worker01.home.local   <none>           <none>
```

サービスを確認する。

```sh
kubectl -n knative-serving get service
```

```text
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                   AGE
activator-service            ClusterIP   10.104.163.99   <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   38m
autoscaler                   ClusterIP   10.96.106.126   <none>        9090/TCP,8008/TCP,8080/TCP                38m
autoscaler-bucket-00-of-01   ClusterIP   10.108.74.110   <none>        8080/TCP                                  38m
controller                   ClusterIP   10.109.23.187   <none>        9090/TCP,8008/TCP                         38m
net-kourier-controller       ClusterIP   10.97.200.124   <none>        18000/TCP,9090/TCP                        33m
webhook                      ClusterIP   10.111.78.35    <none>        9090/TCP,8008/TCP,443/TCP                 38m
```

[DNS サーバ](../gateway/dns.md) に設定を追加する。

```sh
cat > /etc/dnsmasq.d/knative <<EOF
address=/knative.home.local/172.16.0.103
EOF
```

再起動して設定を反映させる。

```sh
systemctl restart dnsmasq
```

設定を確認する。

```sh
getent hosts test.knative.home.local
```

```text
172.16.0.103    test.knative.home.local
```

Knatvie Serving にドメインを設定する。

```sh
kubectl patch configmap/config-domain \
    --namespace knative-serving \
    --type merge \
    --patch '{"data":{"knative.home.local":""}}'
```

```text
configmap/config-domain patched
```

### Knative Eventing

Knative Eventing のカスタムリソース定義を構築する。

```sh
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.19.6/eventing-crds.yaml
```

```text
customresourcedefinition.apiextensions.k8s.io/apiserversources.sources.knative.dev created
customresourcedefinition.apiextensions.k8s.io/brokers.eventing.knative.dev created
customresourcedefinition.apiextensions.k8s.io/channels.messaging.knative.dev created
customresourcedefinition.apiextensions.k8s.io/containersources.sources.knative.dev created
customresourcedefinition.apiextensions.k8s.io/eventpolicies.eventing.knative.dev created
customresourcedefinition.apiextensions.k8s.io/eventtransforms.eventing.knative.dev created
customresourcedefinition.apiextensions.k8s.io/eventtypes.eventing.knative.dev created
customresourcedefinition.apiextensions.k8s.io/integrationsinks.sinks.knative.dev created
customresourcedefinition.apiextensions.k8s.io/integrationsources.sources.knative.dev created
customresourcedefinition.apiextensions.k8s.io/jobsinks.sinks.knative.dev created
customresourcedefinition.apiextensions.k8s.io/parallels.flows.knative.dev created
customresourcedefinition.apiextensions.k8s.io/pingsources.sources.knative.dev created
customresourcedefinition.apiextensions.k8s.io/requestreplies.eventing.knative.dev created
customresourcedefinition.apiextensions.k8s.io/sequences.flows.knative.dev created
customresourcedefinition.apiextensions.k8s.io/sinkbindings.sources.knative.dev created
customresourcedefinition.apiextensions.k8s.io/subscriptions.messaging.knative.dev created
customresourcedefinition.apiextensions.k8s.io/triggers.eventing.knative.dev created
```

Knative Eventing を構築する。

```sh
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.19.6/eventing-core.yaml
```

```text
namespace/knative-eventing created
serviceaccount/eventing-controller created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller-resolver created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller-source-observer created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller-sources-controller created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller-manipulator created
clusterrolebinding.rbac.authorization.k8s.io/eventing-controller-crossnamespace-subscriber created
serviceaccount/job-sink created
clusterrolebinding.rbac.authorization.k8s.io/knative-eventing-job-sink created
serviceaccount/pingsource-mt-adapter created
clusterrolebinding.rbac.authorization.k8s.io/knative-eventing-pingsource-mt-adapter created
serviceaccount/eventing-webhook created
clusterrolebinding.rbac.authorization.k8s.io/eventing-webhook created
rolebinding.rbac.authorization.k8s.io/eventing-webhook created
clusterrolebinding.rbac.authorization.k8s.io/eventing-webhook-resolver created
clusterrolebinding.rbac.authorization.k8s.io/eventing-webhook-podspecable-binding created
configmap/config-br-default-channel created
configmap/config-br-defaults created
configmap/default-ch-webhook created
configmap/config-ping-defaults created
configmap/eventing-integrations-images created
configmap/eventing-transformations-images created
configmap/config-features created
configmap/config-kreference-mapping created
configmap/config-leader-election created
configmap/config-logging created
configmap/config-observability created
configmap/config-sugar created
configmap/config-tracing created
deployment.apps/eventing-controller created
deployment.apps/job-sink created
service/job-sink created
deployment.apps/pingsource-mt-adapter created
horizontalpodautoscaler.autoscaling/eventing-webhook created
poddisruptionbudget.policy/eventing-webhook created
deployment.apps/eventing-webhook created
service/eventing-webhook created
customresourcedefinition.apiextensions.k8s.io/apiserversources.sources.knative.dev configured
customresourcedefinition.apiextensions.k8s.io/brokers.eventing.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/channels.messaging.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/containersources.sources.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/eventpolicies.eventing.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/eventtransforms.eventing.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/eventtypes.eventing.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/integrationsinks.sinks.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/integrationsources.sources.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/jobsinks.sinks.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/parallels.flows.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/pingsources.sources.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/requestreplies.eventing.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/sequences.flows.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/sinkbindings.sources.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/subscriptions.messaging.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/triggers.eventing.knative.dev unchanged
clusterrole.rbac.authorization.k8s.io/addressable-resolver created
clusterrole.rbac.authorization.k8s.io/service-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/serving-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/channel-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/broker-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/flows-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/jobsinks-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/integrationsinks-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/eventtransforms-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/eventing-broker-filter created
clusterrole.rbac.authorization.k8s.io/eventing-broker-ingress created
clusterrole.rbac.authorization.k8s.io/eventing-config-reader created
clusterrole.rbac.authorization.k8s.io/channelable-manipulator created
clusterrole.rbac.authorization.k8s.io/meta-channelable-manipulator created
clusterrole.rbac.authorization.k8s.io/knative-eventing-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-messaging-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-flows-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-sources-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-bindings-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-sinks-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-eventing-namespaced-edit created
clusterrole.rbac.authorization.k8s.io/knative-eventing-namespaced-view created
clusterrole.rbac.authorization.k8s.io/knative-eventing-controller created
clusterrole.rbac.authorization.k8s.io/crossnamespace-subscriber created
clusterrole.rbac.authorization.k8s.io/channel-subscriber created
clusterrole.rbac.authorization.k8s.io/broker-subscriber created
clusterrole.rbac.authorization.k8s.io/knative-eventing-job-sink created
clusterrole.rbac.authorization.k8s.io/knative-eventing-pingsource-mt-adapter created
clusterrole.rbac.authorization.k8s.io/podspecable-binding created
clusterrole.rbac.authorization.k8s.io/builtin-podspecable-binding created
clusterrole.rbac.authorization.k8s.io/source-observer created
clusterrole.rbac.authorization.k8s.io/eventing-sources-source-observer created
clusterrole.rbac.authorization.k8s.io/knative-eventing-sources-controller created
clusterrole.rbac.authorization.k8s.io/knative-eventing-webhook created
role.rbac.authorization.k8s.io/knative-eventing-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.eventing.knative.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.eventing.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.eventing.knative.dev created
secret/eventing-webhook-certs created
mutatingwebhookconfiguration.admissionregistration.k8s.io/sinkbindings.webhook.sources.knative.dev created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n knative-eventing get pod -o wide
```

```text
NAME                                  READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
eventing-controller-ff8cfd969-gxrxk   1/1     Running   0          44s   172.17.51.153    worker02.home.local   <none>           <none>
eventing-webhook-75f4bfcf4-xzc4g      1/1     Running   0          44s   172.17.51.154    worker02.home.local   <none>           <none>
job-sink-584876b655-t5vrn             1/1     Running   0          44s   172.17.255.168   worker01.home.local   <none>           <none>
```

サービスを確認する。

```sh
kubectl -n knative-eventing get service
```

```text
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                   AGE
eventing-webhook   ClusterIP   10.102.220.220   <none>        443/TCP                   100s
job-sink           ClusterIP   10.102.246.19    <none>        80/TCP,443/TCP,9092/TCP   100s
```

インメモリチャネルを構築する。

```{note}
インメモリチャネルは非プロフダクション用
```

```sh
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.19.6/in-memory-channel.yaml
```

```text
serviceaccount/imc-controller created
clusterrolebinding.rbac.authorization.k8s.io/imc-controller created
rolebinding.rbac.authorization.k8s.io/imc-controller created
clusterrolebinding.rbac.authorization.k8s.io/imc-controller-resolver created
serviceaccount/imc-dispatcher created
clusterrolebinding.rbac.authorization.k8s.io/imc-dispatcher created
rolebinding.rbac.authorization.k8s.io/imc-dispatcher-tls-role-binding created
role.rbac.authorization.k8s.io/imc-dispatcher-tls-role created
configmap/config-imc-event-dispatcher created
deployment.apps/imc-controller created
service/inmemorychannel-webhook created
service/imc-dispatcher created
deployment.apps/imc-dispatcher created
customresourcedefinition.apiextensions.k8s.io/inmemorychannels.messaging.knative.dev created
clusterrole.rbac.authorization.k8s.io/imc-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/imc-channelable-manipulator created
clusterrole.rbac.authorization.k8s.io/imc-controller created
clusterrole.rbac.authorization.k8s.io/imc-subscriber created
clusterrole.rbac.authorization.k8s.io/imc-dispatcher created
role.rbac.authorization.k8s.io/knative-inmemorychannel-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/inmemorychannel.eventing.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.inmemorychannel.eventing.knative.dev created
secret/inmemorychannel-webhook-certs created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n knative-eventing get pod
```

```text
NAME                                  READY   STATUS    RESTARTS   AGE
eventing-controller-ff8cfd969-gxrxk   1/1     Running   1          27h
eventing-webhook-75f4bfcf4-xzc4g      1/1     Running   1          27h
imc-controller-c6878df48-zthn8        1/1     Running   0          73s
imc-dispatcher-7c86568d45-bt24b       1/1     Running   0          73s
job-sink-584876b655-t5vrn             1/1     Running   1          27h
```

サービスを確認する。

```sh
kubectl -n knative-eventing get service
```

```text
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
eventing-webhook          ClusterIP   10.102.220.220   <none>        443/TCP                     27h
imc-dispatcher            ClusterIP   10.99.189.168    <none>        80/TCP,443/TCP,9090/TCP     2m59s
inmemorychannel-webhook   ClusterIP   10.98.218.146    <none>        443/TCP,9090/TCP,8008/TCP   2m59s
job-sink                  ClusterIP   10.102.246.19    <none>        80/TCP,443/TCP,9092/TCP     27h
```

マルチテナントブローカを構築する。

```sh
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.19.6/mt-channel-broker.yaml
```

```text
clusterrole.rbac.authorization.k8s.io/knative-eventing-mt-channel-broker-controller created
clusterrole.rbac.authorization.k8s.io/knative-eventing-mt-broker-filter created
role.rbac.authorization.k8s.io/mt-broker-filter created
serviceaccount/mt-broker-filter created
clusterrole.rbac.authorization.k8s.io/knative-eventing-mt-broker-ingress created
role.rbac.authorization.k8s.io/mt-broker-ingress created
serviceaccount/mt-broker-ingress-oidc created
serviceaccount/mt-broker-ingress created
clusterrolebinding.rbac.authorization.k8s.io/eventing-mt-channel-broker-controller created
clusterrolebinding.rbac.authorization.k8s.io/knative-eventing-mt-broker-filter created
rolebinding.rbac.authorization.k8s.io/mt-broker-filter created
clusterrolebinding.rbac.authorization.k8s.io/knative-eventing-mt-broker-ingress created
rolebinding.rbac.authorization.k8s.io/mt-broker-ingress created
deployment.apps/mt-broker-filter created
service/broker-filter created
deployment.apps/mt-broker-ingress created
service/broker-ingress created
deployment.apps/mt-broker-controller created
horizontalpodautoscaler.autoscaling/broker-ingress-hpa created
horizontalpodautoscaler.autoscaling/broker-filter-hpa created
```

すべてのポッドが起動するまで待つ。

```sh
watch kubectl -n knative-eventing get pod
```

```text
NAME                                   READY   STATUS    RESTARTS   AGE
eventing-controller-ff8cfd969-gxrxk    1/1     Running   1          27h
eventing-webhook-75f4bfcf4-xzc4g       1/1     Running   1          27h
imc-controller-c6878df48-zthn8         1/1     Running   0          10m
imc-dispatcher-7c86568d45-bt24b        1/1     Running   0          10m
job-sink-584876b655-t5vrn              1/1     Running   1          27h
mt-broker-controller-9fb9645c4-njj4p   1/1     Running   0          39s
mt-broker-filter-7fb5cffc6b-cvtnf      1/1     Running   0          39s
mt-broker-ingress-796dc65ccd-wk5cb     1/1     Running   0          39s
```

サービスを確認する。

```sh
kubectl -n knative-eventing get service
```

```text
NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
broker-filter             ClusterIP   10.100.207.118   <none>        80/TCP,443/TCP,9092/TCP     95s
broker-ingress            ClusterIP   10.104.165.15    <none>        80/TCP,443/TCP,9092/TCP     95s
eventing-webhook          ClusterIP   10.102.220.220   <none>        443/TCP                     27h
imc-dispatcher            ClusterIP   10.99.189.168    <none>        80/TCP,443/TCP,9090/TCP     11m
inmemorychannel-webhook   ClusterIP   10.98.218.146    <none>        443/TCP,9090/TCP,8008/TCP   11m
job-sink                  ClusterIP   10.102.246.19    <none>        80/TCP,443/TCP,9092/TCP     27h
```

## 動作確認

```{note}
Podman または Docker がインストールされた環境で実施する。
```

Knative Function を作成する。

```sh
kn func create -l go hello
```

```text
Created go function in /root/hello
```

コンテナイメージをビルドする。

```sh
kn func build --registry registry.home.local/kn --verbose
```

```text
Building function image
Still building
Still building
Yes, still building
Don't give up on me
Still building
This is taking a while
🙌 Function built: registry.home.local/kn/hello:latest
```

コンテナイメージを確認する。

```sh
podman images registry.home.local/kn/hello:latest
```

```text
REPOSITORY                    TAG         IMAGE ID      CREATED       SIZE
registry.home.local/kn/hello  latest      1104d1a5b811  45 years ago  48.9 MB
```

ローカルで Knative Function を起動する。

```sh
kn func run
```

```text
function up-to-date. Force rebuild with --build
Running on host port 8080
Initializing HTTP function
listening on http port 8080
```

呼び出す。

```sh
kn func invoke
```

```text
"POST / HTTP/1.1\r\nHost: localhost:8080\r\nAccept-Encoding: gzip\r\nContent-Length: 25\r\nContent-Type: application/json\r\nUser-Agent: Go-http-client/1.1\r\n\r\n{\"message\":\"Hello World\"}"
```

コンテナイメージをデプロイする。

```sh
kn func deploy --registry-insecure
```

```text
function up-to-date. Force rebuild with --build
Pushing function image to the registry "registry.home.local" using the "admin" user credentials
🎯 Creating Triggers on the cluster
✅ Function deployed in namespace "default" and exposed at URL:
   http://hello.default.knative.home.local
```

Kubernetes のリソースを確認する。
ポッドが起動しているが、しばらくすると削除される。

```sh
kubectl get all
```

```text
NAME                                          READY   STATUS    RESTARTS   AGE
pod/hello-00001-deployment-86c8968cb9-dckrk   2/2     Running   0          18s

NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP                                         PORT(S)                                              AGE
service/hello                 ExternalName   <none>          kourier-internal.kourier-system.svc.cluster.local   80/TCP                                               17s
service/hello-00001           ClusterIP      10.96.255.131   <none>                                              80/TCP,443/TCP                                       18s
service/hello-00001-private   ClusterIP      10.106.3.209    <none>                                              80/TCP,443/TCP,9090/TCP,9091/TCP,8022/TCP,8012/TCP   18s
service/kubernetes            ClusterIP      10.96.0.1       <none>                                              443/TCP                                              7d3h

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-00001-deployment   1/1     1            1           18s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-00001-deployment-86c8968cb9   1         1         1       18s
```

呼び出す。

```sh
kn func invoke
```

```text
"POST / HTTP/1.1\r\nHost: hello.default.knative.home.local\r\nAccept-Encoding: gzip\r\nContent-Length: 25\r\nContent-Type: application/json\r\nForwarded: for=172.17.255.167;proto=http\r\nK-Proxy-Request: activator\r\nUser-Agent: Go-http-client/1.1\r\nX-Forwarded-For: 172.17.255.167, 172.17.51.148\r\nX-Forwarded-Proto: http\r\nX-Request-Id: cd6b60bf-e159-4dec-90d5-6b9d1fb39343\r\n\r\n{\"message\":\"Hello World\"}"
```

ロードバランサ経由で実行する。

```sh
curl http://hello.default.knative.home.local
```

```text
"GET / HTTP/1.1\r\nHost: hello.default.knative.home.local\r\nAccept: */*\r\nForwarded: for=172.17.255.167;proto=http\r\nK-Proxy-Request: activator\r\nUser-Agent: curl/8.12.1\r\nX-Forwarded-For: 172.17.255.167, 172.17.51.148\r\nX-Forwarded-Proto: http\r\nX-Request-Id: 6d82eb23-c244-434a-868f-8fb80ba7a89a\r\n\r\n"
```
