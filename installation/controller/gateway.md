# Gateway

## インストール

Ingress Gateway Fabric を構築する。

git をインストールする。

```sh
dnf install -y git-core
```

Gateway API のカスタムリソース定義を作成する。

```sh
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.1.4" | \
    kubectl apply -f -
```

```text
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
```

Nginx Gateway Fabric のカスタムリソース定義を作成する。

```sh
kubectl apply --server-side -f "https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.4/deploy/crds.yaml"
```

```text
customresourcedefinition.apiextensions.k8s.io/clientsettingspolicies.gateway.nginx.org serverside-applied
customresourcedefinition.apiextensions.k8s.io/nginxgateways.gateway.nginx.org serverside-applied
customresourcedefinition.apiextensions.k8s.io/nginxproxies.gateway.nginx.org serverside-applied
customresourcedefinition.apiextensions.k8s.io/observabilitypolicies.gateway.nginx.org serverside-applied
customresourcedefinition.apiextensions.k8s.io/snippetsfilters.gateway.nginx.org serverside-applied
customresourcedefinition.apiextensions.k8s.io/upstreamsettingspolicies.gateway.nginx.org serverside-applied
```

Nginx Gateway Fabric をデプロイする。

```sh
kubectl apply -f "https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.4/deploy/default/deploy.yaml"
```

```text
namespace/nginx-gateway created
serviceaccount/nginx-gateway created
serviceaccount/nginx-gateway-cert-generator created
role.rbac.authorization.k8s.io/nginx-gateway-cert-generator created
clusterrole.rbac.authorization.k8s.io/nginx-gateway created
rolebinding.rbac.authorization.k8s.io/nginx-gateway-cert-generator created
clusterrolebinding.rbac.authorization.k8s.io/nginx-gateway created
service/nginx-gateway created
deployment.apps/nginx-gateway created
job.batch/nginx-gateway-cert-generator created
gatewayclass.gateway.networking.k8s.io/nginx created
nginxgateway.gateway.nginx.org/nginx-gateway-config created
nginxproxy.gateway.nginx.org/nginx-gateway-proxy-config created
```

デプロイを確認する。

```sh
kubectl get all -n nginx-gateway -o wide
```

```text
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
pod/nginx-gateway-6477c87c8-xq9vv   1/1     Running   0          99s   172.17.2.19   controller.home.local   <none>           <none>

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/nginx-gateway   ClusterIP   10.105.48.93   <none>        443/TCP   99s   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS      IMAGES                                     SELECTOR
deployment.apps/nginx-gateway   1/1     1            1           99s   nginx-gateway   ghcr.io/nginx/nginx-gateway-fabric:2.1.4   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                                      DESIRED   CURRENT   READY   AGE   CONTAINERS      IMAGES                                     SELECTOR
replicaset.apps/nginx-gateway-6477c87c8   1         1         1       99s   nginx-gateway   ghcr.io/nginx/nginx-gateway-fabric:2.1.4   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway,pod-template-hash=6477c87c8
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
```

### デバイス

デバイスを確認する。

```sh
ip -d link show cali81f8b902e6e
```

```text
16: cali81f8b902e6e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 68d7e324-8071-40f1-a5b1-30ba188dc649 promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 68d7e324-8071-40f1-a5b1-30ba188dc649 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether b6:e5:41:76:9e:08 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show cali81f8b902e6e
```

```text
16: cali81f8b902e6e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 68d7e324-8071-40f1-a5b1-30ba188dc649
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 68d7e324-8071-40f1-a5b1-30ba188dc649 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether b6:e5:41:76:9e:08 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d
    inet 172.17.2.19/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b4e5:41ff:fe76:9e08/64 scope link proto kernel_ll
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
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 68d7e324-8071-40f1-a5b1-30ba188dc649 ip route show
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
172.17.2.16 dev calif35f658de0d lladdr aa:cc:4a:61:51:80 REACHABLE
172.17.2.12 dev cali93ae5b22e23 lladdr de:47:01:a6:23:36 REACHABLE
172.16.0.254 dev enp1s0 lladdr 52:54:00:65:c6:68 REACHABLE
172.17.2.14 dev cali30782974e56 lladdr 06:7b:37:72:06:f8 REACHABLE
172.17.2.10 dev cali672f36e7b7a lladdr 8a:67:37:08:6e:e1 REACHABLE
```

ネットワーク名前空間内の ARP テーブルを確認する。

```sh
ip netns exec 68d7e324-8071-40f1-a5b1-30ba188dc649 ip neigh
```

```text
169.254.1.1 dev eth0 lladdr ee:ee:ee:ee:ee:ee REACHABLE
```

### nftables

ルールセットを確認する。

```sh
nft list ruleset ip
```

```text
# Warning: table ip mangle is managed by iptables-nft, do not touch!
table ip mangle {
        chain KUBE-IPTABLES-HINT {
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain cali-PREROUTING {
                 ct state related,established counter packets 1540445 bytes 590774021 accept
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 20131 bytes 1208654 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-from-host-endpoint {
        }

        chain cali-POSTROUTING {
                 meta mark & 0x00010000 == 0x00010000 counter packets 25 bytes 2294 return
                 counter packets 1544558 bytes 540657954 meta mark set mark and 0xffe4ffff
                 ct status dnat counter packets 161548 bytes 185126935 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority mangle; policy accept;
                 counter packets 1560576 bytes 591982675 jump cali-PREROUTING
        }

        chain POSTROUTING {
                type filter hook postrouting priority mangle; policy accept;
                 counter packets 1544583 bytes 540660248 jump cali-POSTROUTING
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain KUBE-FIREWALL {
                ip saddr != 127.0.0.0/8 ip daddr 127.0.0.0/8  ct status dnat counter packets 0 bytes 0 drop
        }

        chain OUTPUT {
                type filter hook output priority filter; policy accept;
                 counter packets 1544505 bytes 540643204 jump cali-OUTPUT
                ct state new  counter packets 25883 bytes 1567367 jump KUBE-PROXY-FIREWALL
                ct state new  counter packets 25883 bytes 1567367 jump KUBE-SERVICES
                counter packets 1591521 bytes 567830217 jump KUBE-FIREWALL
        }

        chain INPUT {
                type filter hook input priority filter; policy accept;
                 counter packets 1560476 bytes 591964311 jump cali-INPUT
                ct state new  counter packets 20563 bytes 1233780 jump KUBE-PROXY-FIREWALL
                 counter packets 1548484 bytes 602088953 jump KUBE-NODEPORTS
                ct state new  counter packets 20563 bytes 1233780 jump KUBE-EXTERNAL-SERVICES
                counter packets 1555211 bytes 607230812 jump KUBE-FIREWALL
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-EXTERNAL-SERVICES {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                 counter packets 98 bytes 18260 jump cali-FORWARD
                ct state new  counter packets 29 bytes 2666 jump KUBE-PROXY-FIREWALL
                 counter packets 33 bytes 3338 jump KUBE-FORWARD
                ct state new  counter packets 29 bytes 2666 jump KUBE-SERVICES
                ct state new  counter packets 29 bytes 2666 jump KUBE-EXTERNAL-SERVICES
                  meta mark & 0x00010000 == 0x00010000 counter packets 25 bytes 2294 accept
                 counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain KUBE-NODEPORTS {
        }

        chain KUBE-SERVICES {
        }

        chain KUBE-FORWARD {
                ct state invalid counter packets 0 bytes 0 drop
                 meta mark & 0x00004000 == 0x00004000 counter packets 0 bytes 0 accept
                 ct state related,established counter packets 0 bytes 0 accept
        }

        chain KUBE-PROXY-FIREWALL {
        }

        chain cali-to-hep-forward {
        }

        chain cali-wl-to-host {
                 counter packets 52077 bytes 11178010 jump cali-from-wl-dispatch
                  counter packets 2 bytes 120 accept
        }

        chain cali-OUTPUT {
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                oifname "cali*"  counter packets 46204 bytes 9899676 return
                meta l4proto udp   udp dport 4789 fib saddr type local xt match "set" counter packets 0 bytes 0 accept
                 counter packets 1498301 bytes 530743528 meta mark set mark and 0xffe4ffff
                 ct status dnat counter packets 1361822 bytes 353672520 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-pro-_kJqfZpgUe7r2t4A-14 {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARE0|kns.calico-apiserver" group 2 snaplen 80
        }

        chain cali-pri-kns.calico-system {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARI0|kns.calico-system" group 1 snaplen 80
        }

        chain cali-pri-_nzzjLvInId1gPHmQz_ {
                  counter packets 0 bytes 0
        }

        chain cali-tw-calide6f5a24f1d {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                  counter packets 0 bytes 0 meta mark set mark and 0xfffdffff
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 jump cali-pi-_U7WUiLyTu5Vc3j6v19u
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 log prefix "DPI|default" group 1 snaplen 80
                  meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pri-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_eY4Bnp6m80Op5FOwqd
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-from-wl-dispatch {
                iifname "cali0cd00cf4c0f"  counter packets 0 bytes 0 goto cali-fw-cali0cd00cf4c0f
                iifname "cali30782974e56"  counter packets 752 bytes 133652 goto cali-fw-cali30782974e56
                iifname "cali41393fe08f0"  counter packets 1294 bytes 106165 goto cali-fw-cali41393fe08f0
                iifname "cali53023bf1b9d"  counter packets 0 bytes 0 goto cali-fw-cali53023bf1b9d
                iifname "cali672f36e7b7a"  counter packets 2035 bytes 261795 goto cali-fw-cali672f36e7b7a
                iifname "cali81f8b902e6e"  counter packets 13982 bytes 1956072 goto cali-fw-cali81f8b902e6e
                iifname "cali93ae5b22e23"  counter packets 1292 bytes 105228 goto cali-fw-cali93ae5b22e23
                iifname "calide6f5a24f1d"  counter packets 336 bytes 28243 goto cali-fw-calide6f5a24f1d
                iifname "calif35f658de0d"  counter packets 2503 bytes 1293832 goto cali-fw-calif35f658de0d
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-cali0cd00cf4c0f {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                  counter packets 0 bytes 0 meta mark set mark and 0xfffdffff
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 jump cali-po-_YYnSgB46MA1TYU44kJq
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 log prefix "DPE|default" group 2 snaplen 80
                  meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_jtt6i-KVVwZ-74H4ov
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-_ymJUz7yzI6NOKJhG2- {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali93ae5b22e23 {
                 ct state related,established counter packets 6074 bytes 494667 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 5 bytes 449 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 5 bytes 449 jump cali-pro-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 5 bytes 449 return
                 counter packets 0 bytes 0 jump cali-pro-_u2Tn2rSoAPffvE7JO6
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-tw-cali30782974e56 {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                  counter packets 0 bytes 0 meta mark set mark and 0xfffdffff
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 jump cali-pi-_FDiLImilezd09cpg5ci
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 log prefix "DPI|default" group 1 snaplen 80
                  meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pri-_kJqfZpgUe7r2t4A-14
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_4yi5_iSUAwsU8zMHTk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-tw-cali672f36e7b7a {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_nzzjLvInId1gPHmQz_
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pi-_U7WUiLyTu5Vc3j6v19u {
                meta l4proto tcp   tcp dport 7443 counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "API0|calico-system/knp.default.goldmane" group 1 snaplen 80
        }

        chain cali-po-_YYnSgB46MA1TYU44kJq {
                meta l4proto tcp   xt match "set" tcp dport 7443 counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "APE0|calico-system/knp.default.whisker" group 2 snaplen 80
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                meta l4proto tcp  xt match "set" tcp dport 53 counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "APE1|calico-system/knp.default.whisker" group 2 snaplen 80
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                meta l4proto udp  xt match "set" udp dport 53 counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "APE2|calico-system/knp.default.whisker" group 2 snaplen 80
        }

        chain cali-pro-_4yi5_iSUAwsU8zMHTk {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali41393fe08f0 {
                 ct state related,established counter packets 6031 bytes 490649 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 4 bytes 370 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 4 bytes 370 jump cali-pro-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 4 bytes 370 return
                 counter packets 0 bytes 0 jump cali-pro-_u2Tn2rSoAPffvE7JO6
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-cali53023bf1b9d {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_ymJUz7yzI6NOKJhG2-
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-_nzzjLvInId1gPHmQz_ {
                  counter packets 0 bytes 0
        }

        chain cali-pri-_jtt6i-KVVwZ-74H4ov {
                  counter packets 0 bytes 0
        }

        chain cali-pi-_FDiLImilezd09cpg5ci {
                meta l4proto tcp   tcp dport 5443 counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "API0|calico-apiserver/knp.default.allow-apiserver" group 1 snaplen 80
        }

        chain cali-tw-cali41393fe08f0 {
                 ct state related,established counter packets 6 bytes 909 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 10 bytes 961 meta mark set mark and 0xfffcffff
                 counter packets 10 bytes 961 jump cali-pri-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 10 bytes 961 return
                 counter packets 0 bytes 0 jump cali-pri-_u2Tn2rSoAPffvE7JO6
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-cali30782974e56 {
                 ct state related,established counter packets 3524 bytes 634680 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-_kJqfZpgUe7r2t4A-14
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_4yi5_iSUAwsU8zMHTk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-kns.kube-system {
                  counter packets 9 bytes 819 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 9 bytes 819 log prefix "ARE0|kns.kube-system" group 2 snaplen 80
        }

        chain cali-pro-_u2Tn2rSoAPffvE7JO6 {
                  counter packets 0 bytes 0
        }

        chain cali-pro-_eY4Bnp6m80Op5FOwqd {
                  counter packets 0 bytes 0
        }

        chain cali-fw-calif35f658de0d {
                 ct state related,established counter packets 12267 bytes 6311036 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-_kJqfZpgUe7r2t4A-14
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_4yi5_iSUAwsU8zMHTk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-tw-cali0cd00cf4c0f {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                  counter packets 0 bytes 0 meta mark set mark and 0xfffdffff
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 jump cali-pi-_YYnSgB46MA1TYU44kJq
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 log prefix "DPI|default" group 1 snaplen 80
                  meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pri-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_jtt6i-KVVwZ-74H4ov
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-_kJqfZpgUe7r2t4A-14 {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARI0|kns.calico-apiserver" group 1 snaplen 80
        }

        chain cali-pri-_4yi5_iSUAwsU8zMHTk {
                  counter packets 0 bytes 0
        }

        chain cali-INPUT {
                meta l4proto udp   udp dport 4789 xt match "set" fib daddr type local counter packets 0 bytes 0 accept
                meta l4proto udp   udp dport 4789 fib daddr type local counter packets 0 bytes 0 drop
                iifname "cali*"  counter packets 52077 bytes 11178010 goto cali-wl-to-host
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 1508399 bytes 580786301 meta mark set mark and 0xffe4ffff
                 counter packets 1508399 bytes 580786301 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-from-host-endpoint {
        }

        chain cali-pri-_u2Tn2rSoAPffvE7JO6 {
                  counter packets 0 bytes 0
        }

        chain cali-tw-cali53023bf1b9d {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_ymJUz7yzI6NOKJhG2-
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-calide6f5a24f1d {
                 ct state related,established counter packets 664 bytes 49999 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_eY4Bnp6m80Op5FOwqd
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-to-wl-dispatch {
                oifname "cali0cd00cf4c0f"  counter packets 0 bytes 0 goto cali-tw-cali0cd00cf4c0f
                oifname "cali30782974e56"  counter packets 0 bytes 0 goto cali-tw-cali30782974e56
                oifname "cali41393fe08f0"  counter packets 14 bytes 1522 goto cali-tw-cali41393fe08f0
                oifname "cali53023bf1b9d"  counter packets 0 bytes 0 goto cali-tw-cali53023bf1b9d
                oifname "cali672f36e7b7a"  counter packets 0 bytes 0 goto cali-tw-cali672f36e7b7a
                oifname "cali81f8b902e6e"  counter packets 19 bytes 4751 goto cali-tw-cali81f8b902e6e
                oifname "cali93ae5b22e23"  counter packets 7 bytes 763 goto cali-tw-cali93ae5b22e23
                oifname "calide6f5a24f1d"  counter packets 0 bytes 0 goto cali-tw-calide6f5a24f1d
                oifname "calif35f658de0d"  counter packets 0 bytes 0 goto cali-tw-calif35f658de0d
                  counter packets 0 bytes 0 drop
        }

        chain cali-cidr-block {
        }

        chain cali-pro-_jtt6i-KVVwZ-74H4ov {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali672f36e7b7a {
                 ct state related,established counter packets 9551 bytes 1236636 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_nzzjLvInId1gPHmQz_
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-kns.kube-system {
                  counter packets 15 bytes 1415 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 15 bytes 1415 log prefix "ARI0|kns.kube-system" group 1 snaplen 80
        }

        chain cali-tw-cali93ae5b22e23 {
                 ct state related,established counter packets 8 bytes 1391 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 5 bytes 454 meta mark set mark and 0xfffcffff
                 counter packets 5 bytes 454 jump cali-pri-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 5 bytes 454 return
                 counter packets 0 bytes 0 jump cali-pri-_u2Tn2rSoAPffvE7JO6
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-from-hep-forward {
        }

        chain cali-to-host-endpoint {
        }

        chain cali-pro-kns.calico-system {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARE0|kns.calico-system" group 2 snaplen 80
        }

        chain cali-pri-_eY4Bnp6m80Op5FOwqd {
                  counter packets 0 bytes 0
        }

        chain cali-tw-calif35f658de0d {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                  counter packets 0 bytes 0 meta mark set mark and 0xfffdffff
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 jump cali-pi-_FDiLImilezd09cpg5ci
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 log prefix "DPI|default" group 1 snaplen 80
                  meta mark & 0x00020000 == 0x00000000 counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pri-_kJqfZpgUe7r2t4A-14
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_4yi5_iSUAwsU8zMHTk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-FORWARD {
                 counter packets 98 bytes 18260 meta mark set mark and 0xffe5ffff
                 meta mark & 0x00010000 == 0x00000000 counter packets 98 bytes 18260 jump cali-from-hep-forward
                iifname "cali*"  counter packets 65 bytes 11209 jump cali-from-wl-dispatch
                oifname "cali*"  counter packets 48 bytes 8466 jump cali-to-wl-dispatch
                 counter packets 25 bytes 2294 jump cali-to-hep-forward
                 counter packets 25 bytes 2294 jump cali-cidr-block
        }

        chain cali-pi-_YYnSgB46MA1TYU44kJq {
                  counter packets 0 bytes 0
        }

        chain cali-pri-_ymJUz7yzI6NOKJhG2- {
                  counter packets 0 bytes 0
        }

        chain cali-tw-cali81f8b902e6e {
                 ct state related,established counter packets 19 bytes 4751 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.nginx-gateway
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_VZu5ATptS7dSAoXoNk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-kns.nginx-gateway {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARI0|kns.nginx-gateway" group 1 snaplen 80
        }

        chain cali-pri-_VZu5ATptS7dSAoXoNk {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali81f8b902e6e {
                 ct state related,established counter packets 13965 bytes 1954537 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 17 bytes 1535 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 17 bytes 1535 jump cali-pro-kns.nginx-gateway
                  meta mark & 0x00010000 == 0x00010000 counter packets 17 bytes 1535 return
                 counter packets 0 bytes 0 jump cali-pro-_VZu5ATptS7dSAoXoNk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-kns.nginx-gateway {
                  counter packets 17 bytes 1535 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 17 bytes 1535 log prefix "ARE0|kns.nginx-gateway" group 2 snaplen 80
        }

        chain cali-pro-_VZu5ATptS7dSAoXoNk {
                  counter packets 0 bytes 0
        }
}
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-SERVICES {
                meta l4proto tcp ip daddr 10.96.0.1  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-NPX46M4PTMTKRN6Y
                meta l4proto udp ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-TCOU7JCQXEZGVUNU
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-ERIFXISQEP7F7OF4
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-SVC-JD5MR3NA4I4DYORP
                meta l4proto tcp ip daddr 10.111.96.199  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-I24EZXP75AX5E7TU
                meta l4proto tcp ip daddr 10.107.225.207  tcp dport 7443 counter packets 0 bytes 0 jump KUBE-SVC-RMIM2MJTF3UE534N
                meta l4proto tcp ip daddr 10.98.161.214  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-SVC-RK657RLKDNVNU64O
                meta l4proto tcp ip daddr 10.104.39.6  tcp dport 8081 counter packets 0 bytes 0 jump KUBE-SVC-UA7HCJMIMBJJDU4H
                meta l4proto tcp ip daddr 10.105.48.93  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-SEU3WZYSL6OBXFRO
                 fib daddr type local counter packets 4248 bytes 254880 jump KUBE-NODEPORTS
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                 counter packets 24483 bytes 1475705 jump cali-OUTPUT
                 counter packets 25066 bytes 1514399 jump KUBE-SERVICES
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                 counter packets 28 bytes 2474 jump cali-PREROUTING
                 counter packets 42 bytes 3446 jump KUBE-SERVICES
        }

        chain KUBE-POSTROUTING {
                meta mark & 0x00004000 != 0x00004000 counter packets 6109 bytes 367746 return
                counter packets 0 bytes 0 meta mark set mark xor 0x4000
                 counter packets 0 bytes 0 masquerade fully-random
        }

        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                 counter packets 24997 bytes 1508529 jump KUBE-POSTROUTING
                 counter packets 24440 bytes 1472527 jump cali-POSTROUTING
        }

        chain KUBE-NODEPORTS {
        }

        chain KUBE-MARK-MASQ {
                counter packets 0 bytes 0 meta mark set mark or 0x4000
        }

        chain KUBE-SVC-NPX46M4PTMTKRN6Y {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.1  tcp dport 443 counter packets 6 bytes 360 jump KUBE-MARK-MASQ
                 counter packets 8 bytes 480 jump KUBE-SEP-23Y66C2VAJ3WDEMI
        }

        chain KUBE-SEP-23Y66C2VAJ3WDEMI {
                ip saddr 172.16.0.11  counter packets 6 bytes 360 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 8 bytes 480 dnat to 172.16.0.11:6443
        }

        chain KUBE-SVC-TCOU7JCQXEZGVUNU {
                meta l4proto udp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 5 bytes 454 jump KUBE-SEP-5FL5ISI2YRT6Y7BH
                 counter packets 10 bytes 961 jump KUBE-SEP-UHAP4MO6KX56B2GW
        }

        chain KUBE-SEP-5FL5ISI2YRT6Y7BH {
                ip saddr 172.17.2.12  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp  meta l4proto udp counter packets 5 bytes 454 dnat to 172.17.2.12:53
        }

        chain KUBE-SVC-ERIFXISQEP7F7OF4 {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-MOTGC5ZSJDYEK44E
                 counter packets 0 bytes 0 jump KUBE-SEP-NZCOKNBUF6VLEG7M
        }

        chain KUBE-SEP-MOTGC5ZSJDYEK44E {
                ip saddr 172.17.2.12  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.12:53
        }

        chain KUBE-SVC-JD5MR3NA4I4DYORP {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-YES4TO7LVIGUXIUO
                 counter packets 0 bytes 0 jump KUBE-SEP-UNOTEOHYEEQBAMPD
        }

        chain KUBE-SEP-YES4TO7LVIGUXIUO {
                ip saddr 172.17.2.12  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.12:9153
        }

        chain KUBE-SEP-NZCOKNBUF6VLEG7M {
                ip saddr 172.17.2.15  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.15:53
        }

        chain KUBE-SEP-UNOTEOHYEEQBAMPD {
                ip saddr 172.17.2.15  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.15:9153
        }

        chain KUBE-SEP-UHAP4MO6KX56B2GW {
                ip saddr 172.17.2.15  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp  meta l4proto udp counter packets 10 bytes 961 dnat to 172.17.2.15:53
        }

        chain KUBE-SVC-UA7HCJMIMBJJDU4H {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.104.39.6  tcp dport 8081 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-V3NSGPAOEZAZASIS
        }

        chain KUBE-SEP-V3NSGPAOEZAZASIS {
                ip saddr 172.17.2.13  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.13:8081
        }

        chain KUBE-SVC-RK657RLKDNVNU64O {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.98.161.214  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-T7B34ACBXBTVP7YI
        }

        chain KUBE-SEP-T7B34ACBXBTVP7YI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.16.0.11:5473
        }

        chain cali-PREROUTING {
                 counter packets 28 bytes 2474 jump cali-fip-dnat
        }

        chain cali-fip-dnat {
        }

        chain cali-POSTROUTING {
                 counter packets 24440 bytes 1472527 jump cali-fip-snat
                 counter packets 24440 bytes 1472527 jump cali-nat-outgoing
                oifname "vxlan.calico"  fib saddr . oif type != local fib saddr type local counter packets 0 bytes 0 masquerade fully-random
        }

        chain cali-fip-snat {
        }

        chain cali-nat-outgoing {
                 xt match "set" xt match "set" counter packets 10 bytes 879 masquerade fully-random
        }

        chain cali-OUTPUT {
                 counter packets 24483 bytes 1475705 jump cali-fip-dnat
        }

        chain KUBE-SVC-RMIM2MJTF3UE534N {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.107.225.207  tcp dport 7443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-UZGLWGTNAGZINNAV
        }

        chain KUBE-SEP-UZGLWGTNAGZINNAV {
                ip saddr 172.17.2.17  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.17:7443
        }

        chain KUBE-SVC-I24EZXP75AX5E7TU {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.111.96.199  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-ALWVLIDVD2HVXGAQ
                 counter packets 0 bytes 0 jump KUBE-SEP-GTOCSUPK2KFVQLBE
        }

        chain KUBE-SEP-GTOCSUPK2KFVQLBE {
                ip saddr 172.17.2.16  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.16:5443
        }

        chain KUBE-SEP-ALWVLIDVD2HVXGAQ {
                ip saddr 172.17.2.14  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.14:5443
        }

        chain KUBE-SVC-SEU3WZYSL6OBXFRO {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.105.48.93  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-LGHLGACKMLQNACGS
        }

        chain KUBE-SEP-LGHLGACKMLQNACGS {
                ip saddr 172.17.2.19  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.19:8443
        }
}
# Warning: table ip raw is managed by iptables-nft, do not touch!
table ip raw {
        chain cali-PREROUTING {
                 counter packets 1560576 bytes 591982675 meta mark set mark and 0xffe4ffff
                meta l4proto udp  udp dport 4789 counter packets 0 bytes 0 notrack
                iifname "cali*"  counter packets 52142 bytes 11189219 meta mark set mark or 0x80000
                 meta mark & 0x00080000 == 0x00080000 counter packets 52142 bytes 11189219 jump cali-rpf-skip
                 meta mark & 0x00080000 == 0x00080000 fib saddr . mark . iif oif 0 counter packets 0 bytes 0 drop
                 meta mark & 0x00080000 == 0x00000000 counter packets 1508434 bytes 580793456 jump cali-from-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-rpf-skip {
        }

        chain cali-from-host-endpoint {
        }

        chain cali-OUTPUT {
                 counter packets 1544507 bytes 540643308 meta mark set mark and 0xffe4ffff
                 counter packets 1544507 bytes 540643308 jump cali-to-host-endpoint
                meta l4proto udp  udp dport 4789 counter packets 0 bytes 0 notrack
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority raw; policy accept;
                 counter packets 1560576 bytes 591982675 jump cali-PREROUTING
        }

        chain OUTPUT {
                type filter hook output priority raw; policy accept;
                 counter packets 1544507 bytes 540643308 jump cali-OUTPUT
        }
}
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n nginx-gateway -o wide
```

```text
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
pod/nginx-gateway-6477c87c8-xq9vv   1/1     Running   0          15m   172.17.2.19   controller.home.local   <none>           <none>

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/nginx-gateway   ClusterIP   10.105.48.93   <none>        443/TCP   15m   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS      IMAGES                                     SELECTOR
deployment.apps/nginx-gateway   1/1     1            1           15m   nginx-gateway   ghcr.io/nginx/nginx-gateway-fabric:2.1.4   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway

NAME                                      DESIRED   CURRENT   READY   AGE   CONTAINERS      IMAGES                                     SELECTOR
replicaset.apps/nginx-gateway-6477c87c8   1         1         1       15m   nginx-gateway   ghcr.io/nginx/nginx-gateway-fabric:2.1.4   app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway,pod-template-hash=6477c87c8
```

```sh
kubectl describe service nginx-gateway -n nginx-gateway
```

```text
Name:                     nginx-gateway
Namespace:                nginx-gateway
Labels:                   app.kubernetes.io/instance=nginx-gateway
                          app.kubernetes.io/name=nginx-gateway
                          app.kubernetes.io/version=2.1.4
Annotations:              <none>
Selector:                 app.kubernetes.io/instance=nginx-gateway,app.kubernetes.io/name=nginx-gateway
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.105.48.93
IPs:                      10.105.48.93
Port:                     agent-grpc  443/TCP
TargetPort:               8443/TCP
Endpoints:                172.17.2.19:8443
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
CONTAINER           IMAGE                                                              CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                             NAMESPACE
35104e7c5aadc       44c116ec3db7fbb99aafe27302134b6de6cced67a71c7d8fd7d7dfa77cfc242f   16 minutes ago      Running             nginx-gateway               0                   1e132d225a3c2       nginx-gateway-6477c87c8-xq9vv                   nginx-gateway
7e86e0b89f60e       a7d029fd8f6be94c26af980675c1650818e1e6e19dbd2f8c13e6e61963f021e8   2 hours ago         Running             goldmane                    2                   b6e6d502bdf37       goldmane-85c8f6d476-qckwz                       calico-system
a4efeced770df       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a   2 hours ago         Running             calico-apiserver            2                   b9a9f507e4213       calico-apiserver-5c49f7b8cd-5thsd               calico-apiserver
4be35211a40b9       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b   2 hours ago         Running             coredns                     2                   ed7589b8ee54a       coredns-674b8bbfcf-csznq                        kube-system
fa2c52c47e221       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a   2 hours ago         Running             calico-apiserver            2                   2303f2c3ebed6       calico-apiserver-5c49f7b8cd-cw8rw               calico-apiserver
2c32eaa41ddbb       7e29b0984d517678aab6ca138482c318989f6f28daf9d3b5dd6e4a5a3115ac16   2 hours ago         Running             whisker-backend             2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
dabfc11320849       9a4eedeed4a531acefb7f5d0a1b7e3856b1a9a24d9e7d25deef2134d7a734c2d   2 hours ago         Running             whisker                     2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
427dc8955c1dc       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b   2 hours ago         Running             coredns                     2                   5c91d08fb137b       coredns-674b8bbfcf-ng4p7                        kube-system
c883af54c988d       b8f31c4fdaed3fa08af64de3d37d65a4c2ea0d9f6f522cb60d2e0cb424f8dd8a   2 hours ago         Running             csi-node-driver-registrar   2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
c99ee13428df9       666f4e02e75c30547109a06ed75b415a990a970811173aa741379cfaac4d9dd7   2 hours ago         Running             calico-csi                  2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
8b259a4736bf6       df191a54fb79de3c693f8b1b864a1bd3bd14f63b3fff9d5fa4869c471ce3cd37   2 hours ago         Running             calico-kube-controllers     2                   6fb73445c5b3a       calico-kube-controllers-98fbcc856-vgxhq         calico-system
b0bb7e3a28a19       ce9c4ac0f175f22c56e80844e65379d9ebe1d8a4e2bbb38dc1db0f53a8826f0f   2 hours ago         Running             calico-node                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
2d63dd1aa09a5       034822460c2f667e1f4a7679c843cc35ce1bf2c25dec86f04e07fb403df7e458   2 hours ago         Exited              install-cni                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
dbbc87a2e095e       4f2b088ed6fdfc6a97ac0650a4ba8171107d6656ce265c592e4c8423fd10e5c4   2 hours ago         Exited              flexvol-driver              2                   401470c9b90cf       calico-node-7lw6s                               calico-system
5c60b077e08fd       1911afdd8478c6ca3036ff85614050d5d19acc0f0c3f6a5a7b3e34b38dd309c9   2 hours ago         Running             tigera-operator             2                   b00e4ea97c8cd       tigera-operator-755d956888-2l5ks                tigera-operator
ce2a879a71387       2844ee7bb56c2c194e1f4adafb9e7b60b9ed16aa4d07ab8ad1f019362e2efab3   2 hours ago         Running             kube-proxy                  2                   862ab4aed1086       kube-proxy-xn5ft                                kube-system
55ad4fde21a9f       1d7bb7b0cce2924d35c7c26f6b6600409ea7c9535074c3d2e517ffbb3a0e0b36   2 hours ago         Running             calico-typha                2                   4d2521ead8ea9       calico-typha-6574bcdfc5-fgdgz                   calico-system
c64f172909e7e       33b680aadf474b7e5e73957fc00c6af86dd0484c699c8461ba33ee656d1823bf   2 hours ago         Running             kube-scheduler              3                   0246fe2061ab7       kube-scheduler-controller.home.local            kube-system
4d4ad99f7cadb       8bb43160a0df4d7d34c89d9edbc48735bc2f830771e4b501937338221be0f668   2 hours ago         Running             kube-controller-manager     3                   661ffd4ff1052       kube-controller-manager-controller.home.local   kube-system
886b87e7b5891       b7335a56022aba291f5df653c01b7ab98d64fb5cab221378617f4a1236e06a62   2 hours ago         Running             kube-apiserver              3                   f17ac77d88354       kube-apiserver-controller.home.local            kube-system
c75beff2be6a1       499038711c0816eda03a1ad96a8eb0440c005baa6949698223c6176b7f5077e1   2 hours ago         Running             etcd                        3                   0b701b25c64e8       etcd-controller.home.local                      kube-system
```

nginx-gateway が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 35104e7c5aadc | jq '.info.pid')
```

```text
68d7e324-8071-40f1-a5b1-30ba188dc649
```

コンテナイメージを確認する。

```sh
crictl images
```

```text
IMAGE                                     TAG                 IMAGE ID            SIZE
docker.io/calico/apiserver                v3.30.3             879f2443aed05       118MB
docker.io/calico/cni                      v3.30.3             034822460c2f6       162MB
docker.io/calico/csi                      v3.30.3             666f4e02e75c3       20.6MB
docker.io/calico/goldmane                 v3.30.3             a7d029fd8f6be       151MB
docker.io/calico/kube-controllers         v3.30.3             df191a54fb79d       122MB
docker.io/calico/node-driver-registrar    v3.30.3             b8f31c4fdaed3       33.3MB
docker.io/calico/node                     v3.30.3             ce9c4ac0f175f       403MB
docker.io/calico/pod2daemon-flexvol       v3.30.3             4f2b088ed6fdf       12.1MB
docker.io/calico/typha                    v3.30.3             1d7bb7b0cce29       85.2MB
docker.io/calico/whisker-backend          v3.30.3             7e29b0984d517       75.5MB
docker.io/calico/whisker                  v3.30.3             9a4eedeed4a53       14.2MB
ghcr.io/nginx/nginx-gateway-fabric        2.1.4               44c116ec3db7f       50.8MB
quay.io/tigera/operator                   v1.38.6             1911afdd8478c       87.2MB
registry.k8s.io/coredns/coredns           v1.12.0             1cf5f116067c6       71.2MB
registry.k8s.io/etcd                      3.5.21-0            499038711c081       154MB
registry.k8s.io/kube-apiserver            v1.33.5             b7335a56022ab       103MB
registry.k8s.io/kube-controller-manager   v1.33.5             8bb43160a0df4       95.8MB
registry.k8s.io/kube-proxy                v1.33.5             2844ee7bb56c2       99.3MB
registry.k8s.io/kube-scheduler            v1.33.5             33b680aadf474       74.6MB
registry.k8s.io/pause                     3.10                873ed75102791       742kB
```

## 動作確認

[ロードバランサ](./loadbalancer.md) を構築したあとでポッドを作成する。

```sh
kubectl create deployment demo-gateway --image=httpd --port=80
```

```text
deployment.apps/demo-gateway created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```text
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
demo-gateway-848c7cbd64-kszw7   1/1     Running   0          6s    172.17.2.23   controller.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo-gateway
```

```text
service/demo-gateway exposed
```

サービスを確認する。

```sh
kubectl get service
```

```text
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
demo-gateway   ClusterIP   10.106.69.110   <none>        80/TCP    6s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP   3h28m
```

Gateway を作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: demo
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-gateway
spec:
  parentRefs:
  - name: demo
  hostnames:
  - demo-gateway.localdev.me
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: demo-gateway
      port: 80
EOF
```

```text
gateway.gateway.networking.k8s.io/demo created
httproute.gateway.networking.k8s.io/demo-gateway created
```

Gateway を確認する。

```sh
kubectl get gateway.gateway.networking.k8s.io
```

```text
NAME   CLASS   ADDRESS        PROGRAMMED   AGE
demo   nginx   172.16.0.100   True         9s
```

ルーティングを確認する。

```sh
kubectl get httproute.gateway.networking.k8s.io
```

```text
NAME           HOSTNAMES                      AGE
demo-gateway   ["demo-gateway.localdev.me"]   23s
```

接続確認する。

```sh
curl --resolve demo-gateway.localdev.me:80:172.16.0.100 http://demo-gateway.localdev.me/
```

```text
<html><body><h1>It works!</h1></body></html>
```
