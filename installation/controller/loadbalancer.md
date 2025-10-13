# ロードバランサ

ロードバランサとして MetalLB を構築する。

## ファイアウォール

すべてのノードで ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=7946/{tcp,udp}
firewall-cmd --permanent --zone=internal --add-port=7946/{tcp,udp}
firewall-cmd --reload
```

## インストール

MetalLB を構築する。

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml
```

```text
namespace/metallb-system created
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io created
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io created
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/servicebgpstatuses.metallb.io created
customresourcedefinition.apiextensions.k8s.io/servicel2statuses.metallb.io created
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
configmap/metallb-excludel2 created
secret/metallb-webhook-cert created
service/metallb-webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration created
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods -n metallb-system
```

```text
NAME                          READY   STATUS    RESTARTS   AGE
controller-58fdf44d87-tgp79   1/1     Running   0          46s
speaker-n7prq                 1/1     Running   0          46s
```

## 環境確認

Controller Node でネットワーク構成を確認する。

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```text
9aff30be-081c-4026-a352-fbac45faf19d
8bed4c5b-8143-43b6-ac03-a954a9266cf9
7a76ca1d-6c90-42c3-a2ca-e65ea6467c6d
295825a4-7a49-4339-b3da-f568dfa228a1
4e111709-7866-412d-a22f-40218b3826ad
5d787ef5-0bdb-41d0-8241-c330cca02e2a
5f3859db-54a2-4161-a2c3-82e9584b261d
40d96ff5-da8b-44fe-8f07-a6296a84b937
22c34064-9102-41e8-bc25-000396798604 (id: 0)
239ffac8-1209-43ea-a2d3-ad19b8d8848a (id: 1)
4481b8cf-1060-4f55-a40c-3189d75a9654 (id: 2)
781f764e-3969-44c4-9e3d-13fe5e2bc7c4 (id: 3)
2b3f4279-9dac-4dc3-a46c-57c622e06df6 (id: 4)
d63358e4-c916-44cb-a9f6-0e16d885bdab (id: 5)
018b7903-2187-4cb7-8695-42fc1c70cc6f (id: 6)
8be1b599-9424-4733-aea4-36b845182d33 (id: 7)
68d7e324-8071-40f1-a5b1-30ba188dc649 (id: 8)
874871f4-9c7b-4124-b716-f032f5e83e45 (id: 9)
f451a269-aa07-4f03-b326-0fe4892baf08
```

### デバイス

デバイスを確認する。

```sh
ip -d link show cali0817bf63151
```

```text
19: cali0817bf63151@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 874871f4-9c7b-4124-b716-f032f5e83e45 promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 874871f4-9c7b-4124-b716-f032f5e83e45 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0e:d8:ec:61:04:06 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show cali0817bf63151
```

```text
19: cali0817bf63151@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 874871f4-9c7b-4124-b716-f032f5e83e45
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 874871f4-9c7b-4124-b716-f032f5e83e45 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 0e:d8:ec:61:04:06 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d
    inet 172.17.2.22/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::cd8:ecff:fe61:406/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

### ルート

ルーティングを確認する。

```sh
ip route show
```

```text
default via 172.16.0.254 dev enp1s0 proto static metric 100
10.0.0.0/24 dev enp2s0 proto kernel scope link src 10.0.0.11 metric 101
172.16.0.0/24 dev enp1s0 proto kernel scope link src 172.16.0.11 metric 100
blackhole 172.17.2.0/26 proto 80
172.17.2.10 dev cali672f36e7b7a scope link
172.17.2.11 dev cali53023bf1b9d scope link
172.17.2.12 dev cali93ae5b22e23 scope link
172.17.2.13 dev cali0cd00cf4c0f scope link
172.17.2.14 dev cali30782974e56 scope link
172.17.2.15 dev cali41393fe08f0 scope link
172.17.2.16 dev calif35f658de0d scope link
172.17.2.17 dev calide6f5a24f1d scope link
172.17.2.19 dev cali81f8b902e6e scope link
172.17.2.22 dev cali0817bf63151 scope link
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 874871f4-9c7b-4124-b716-f032f5e83e45 ip route show
```

```text
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
```

### ARP テーブル

ARP テーブルを確認する。

```sh
ip neigh
```

```text
172.17.2.15 dev cali41393fe08f0 lladdr 6a:a6:1d:2d:0a:b7 REACHABLE
172.16.0.199 dev enp1s0 lladdr fe:54:00:49:41:b8 REACHABLE
172.17.2.19 dev cali81f8b902e6e lladdr b6:e5:41:76:9e:08 REACHABLE
172.17.2.17 dev calide6f5a24f1d lladdr da:2f:c2:fc:35:10 REACHABLE
172.17.2.22 dev cali0817bf63151 lladdr 0e:d8:ec:61:04:06 REACHABLE
172.17.2.16 dev calif35f658de0d lladdr aa:cc:4a:61:51:80 REACHABLE
172.17.2.12 dev cali93ae5b22e23 lladdr de:47:01:a6:23:36 REACHABLE
172.16.0.254 dev enp1s0 lladdr 52:54:00:65:c6:68 REACHABLE
172.17.2.14 dev cali30782974e56 lladdr 06:7b:37:72:06:f8 REACHABLE
172.17.2.10 dev cali672f36e7b7a lladdr 8a:67:37:08:6e:e1 REACHABLE
```

ネットワーク名前空間内の ARP テーブルを確認する。

```sh
ip netns exec 874871f4-9c7b-4124-b716-f032f5e83e45 ip neigh
```

```text
169.254.1.1 dev eth0 lladdr ee:ee:ee:ee:ee:ee REACHABLE
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n metallb-system -o wide
```

```text
NAME                              READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
pod/controller-58fdf44d87-tgp79   1/1     Running   0          76s   172.17.2.22   controller.home.local   <none>           <none>
pod/speaker-n7prq                 1/1     Running   0          76s   172.16.0.11   controller.home.local   <none>           <none>

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/metallb-webhook-service   ClusterIP   10.99.161.198   <none>        443/TCP   76s   component=controller

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS   IMAGES                            SELECTOR
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   76s   speaker      quay.io/metallb/speaker:v0.15.2   app=metallb,component=speaker

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                               SELECTOR
deployment.apps/controller   1/1     1            1           76s   controller   quay.io/metallb/controller:v0.15.2   app=metallb,component=controller

NAME                                    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                               SELECTOR
replicaset.apps/controller-58fdf44d87   1         1         1       76s   controller   quay.io/metallb/controller:v0.15.2   app=metallb,component=controller,pod-template-hash=58fdf44d87
```

```sh
kubectl describe service metallb-webhook-service -n metallb-system
```

```text
Name:                     metallb-webhook-service
Namespace:                metallb-system
Labels:                   <none>
Annotations:              <none>
Selector:                 component=controller
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.99.161.198
IPs:                      10.99.161.198
Port:                     <unset>  443/TCP
TargetPort:               9443/TCP
Endpoints:                172.17.2.22:9443
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```text
CONTAINER           IMAGE                                                                                                CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                             NAMESPACE
6a9e197a4b8a5       quay.io/metallb/speaker@sha256:7dbe5e4b1ee3be1638b1620e2670649e9900768972060a31d2e809855e31e344      3 minutes ago       Running             speaker                     0                   d04db24e8f2fe       speaker-n7prq                                   metallb-system
7f31331627784       quay.io/metallb/controller@sha256:9bd30b4ee165ad6bababa6334594d854b8c1037c385af83832fb07a4b2c23212   4 minutes ago       Running             controller                  0                   88e1eacb0618f       controller-58fdf44d87-tgp79                     metallb-system
35104e7c5aadc       44c116ec3db7fbb99aafe27302134b6de6cced67a71c7d8fd7d7dfa77cfc242f                                     About an hour ago   Running             nginx-gateway               0                   1e132d225a3c2       nginx-gateway-6477c87c8-xq9vv                   nginx-gateway
7e86e0b89f60e       a7d029fd8f6be94c26af980675c1650818e1e6e19dbd2f8c13e6e61963f021e8                                     2 hours ago         Running             goldmane                    2                   b6e6d502bdf37       goldmane-85c8f6d476-qckwz                       calico-system
a4efeced770df       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a                                     2 hours ago         Running             calico-apiserver            2                   b9a9f507e4213       calico-apiserver-5c49f7b8cd-5thsd               calico-apiserver
4be35211a40b9       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b                                     2 hours ago         Running             coredns                     2                   ed7589b8ee54a       coredns-674b8bbfcf-csznq                        kube-system
fa2c52c47e221       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a                                     2 hours ago         Running             calico-apiserver            2                   2303f2c3ebed6       calico-apiserver-5c49f7b8cd-cw8rw               calico-apiserver
2c32eaa41ddbb       7e29b0984d517678aab6ca138482c318989f6f28daf9d3b5dd6e4a5a3115ac16                                     2 hours ago         Running             whisker-backend             2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
dabfc11320849       9a4eedeed4a531acefb7f5d0a1b7e3856b1a9a24d9e7d25deef2134d7a734c2d                                     2 hours ago         Running             whisker                     2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
427dc8955c1dc       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b                                     2 hours ago         Running             coredns                     2                   5c91d08fb137b       coredns-674b8bbfcf-ng4p7                        kube-system
c883af54c988d       b8f31c4fdaed3fa08af64de3d37d65a4c2ea0d9f6f522cb60d2e0cb424f8dd8a                                     2 hours ago         Running             csi-node-driver-registrar   2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
c99ee13428df9       666f4e02e75c30547109a06ed75b415a990a970811173aa741379cfaac4d9dd7                                     2 hours ago         Running             calico-csi                  2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
8b259a4736bf6       df191a54fb79de3c693f8b1b864a1bd3bd14f63b3fff9d5fa4869c471ce3cd37                                     2 hours ago         Running             calico-kube-controllers     2                   6fb73445c5b3a       calico-kube-controllers-98fbcc856-vgxhq         calico-system
b0bb7e3a28a19       ce9c4ac0f175f22c56e80844e65379d9ebe1d8a4e2bbb38dc1db0f53a8826f0f                                     2 hours ago         Running             calico-node                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
2d63dd1aa09a5       034822460c2f667e1f4a7679c843cc35ce1bf2c25dec86f04e07fb403df7e458                                     2 hours ago         Exited              install-cni                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
dbbc87a2e095e       4f2b088ed6fdfc6a97ac0650a4ba8171107d6656ce265c592e4c8423fd10e5c4                                     2 hours ago         Exited              flexvol-driver              2                   401470c9b90cf       calico-node-7lw6s                               calico-system
5c60b077e08fd       1911afdd8478c6ca3036ff85614050d5d19acc0f0c3f6a5a7b3e34b38dd309c9                                     2 hours ago         Running             tigera-operator             2                   b00e4ea97c8cd       tigera-operator-755d956888-2l5ks                tigera-operator
ce2a879a71387       2844ee7bb56c2c194e1f4adafb9e7b60b9ed16aa4d07ab8ad1f019362e2efab3                                     2 hours ago         Running             kube-proxy                  2                   862ab4aed1086       kube-proxy-xn5ft                                kube-system
55ad4fde21a9f       1d7bb7b0cce2924d35c7c26f6b6600409ea7c9535074c3d2e517ffbb3a0e0b36                                     2 hours ago         Running             calico-typha                2                   4d2521ead8ea9       calico-typha-6574bcdfc5-fgdgz                   calico-system
c64f172909e7e       33b680aadf474b7e5e73957fc00c6af86dd0484c699c8461ba33ee656d1823bf                                     2 hours ago         Running             kube-scheduler              3                   0246fe2061ab7       kube-scheduler-controller.home.local            kube-system
4d4ad99f7cadb       8bb43160a0df4d7d34c89d9edbc48735bc2f830771e4b501937338221be0f668                                     2 hours ago         Running             kube-controller-manager     3                   661ffd4ff1052       kube-controller-manager-controller.home.local   kube-system
886b87e7b5891       b7335a56022aba291f5df653c01b7ab98d64fb5cab221378617f4a1236e06a62                                     2 hours ago         Running             kube-apiserver              3                   f17ac77d88354       kube-apiserver-controller.home.local            kube-system
c75beff2be6a1       499038711c0816eda03a1ad96a8eb0440c005baa6949698223c6176b7f5077e1                                     2 hours ago         Running             etcd                        3                   0b701b25c64e8       etcd-controller.home.local                      kube-system
```

speaker が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 6a9e197a4b8a5 | jq '.info.pid')
```

```text
f451a269-aa07-4f03-b326-0fe4892baf08
```

controller が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 7f31331627784 | jq '.info.pid')
```

```text
874871f4-9c7b-4124-b716-f032f5e83e45
```

コンテナイメージを確認する。

```sh
crictl images
```

```text
IMAGE                                      TAG                 IMAGE ID            SIZE
docker.io/calico/apiserver                 v3.30.3             879f2443aed05       118MB
docker.io/calico/cni                       v3.30.3             034822460c2f6       162MB
docker.io/calico/csi                       v3.30.3             666f4e02e75c3       20.6MB
docker.io/calico/goldmane                  v3.30.3             a7d029fd8f6be       151MB
docker.io/calico/kube-controllers          v3.30.3             df191a54fb79d       122MB
docker.io/calico/node-driver-registrar     v3.30.3             b8f31c4fdaed3       33.3MB
docker.io/calico/node                      v3.30.3             ce9c4ac0f175f       403MB
docker.io/calico/pod2daemon-flexvol        v3.30.3             4f2b088ed6fdf       12.1MB
docker.io/calico/typha                     v3.30.3             1d7bb7b0cce29       85.2MB
docker.io/calico/whisker-backend           v3.30.3             7e29b0984d517       75.5MB
docker.io/calico/whisker                   v3.30.3             9a4eedeed4a53       14.2MB
docker.io/library/httpd                    latest              2416cb32cb59e       120MB
ghcr.io/nginx/nginx-gateway-fabric/nginx   2.1.4               99119d5762c69       155MB
ghcr.io/nginx/nginx-gateway-fabric         2.1.4               44c116ec3db7f       50.8MB
quay.io/metallb/controller                 v0.15.2             1b258f39a2a12       78.3MB
quay.io/metallb/speaker                    v0.15.2             1eeb60bd05864       140MB
quay.io/tigera/operator                    v1.38.6             1911afdd8478c       87.2MB
registry.k8s.io/coredns/coredns            v1.12.0             1cf5f116067c6       71.2MB
registry.k8s.io/etcd                       3.5.21-0            499038711c081       154MB
registry.k8s.io/kube-apiserver             v1.33.5             b7335a56022ab       103MB
registry.k8s.io/kube-controller-manager    v1.33.5             8bb43160a0df4       95.8MB
registry.k8s.io/kube-proxy                 v1.33.5             2844ee7bb56c2       99.3MB
registry.k8s.io/kube-scheduler             v1.33.5             33b680aadf474       74.6MB
registry.k8s.io/pause                      3.10                873ed75102791       742kB
```

## IP プール

ロードバランサに割り当てる IP アドレスを追加する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: public-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.16.0.100-172.16.0.149
EOF
```

```text
ipaddresspool.metallb.io/public-pool created
```

IP アドレスを L2 モードでアドバタイズする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: public
  namespace: metallb-system
spec:
  ipAddressPools:
  - public-pool
EOF
```

```text
l2advertisement.metallb.io/public created
```
