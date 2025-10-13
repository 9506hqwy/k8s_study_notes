# Ingress Controller

## インストール

Ingress NGINX Controller を構築する。

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.3/deploy/static/provider/cloud/deploy.yaml
```

```text
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods -n ingress-nginx
```

```text
NAME                                       READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-7f894db6f-kcb92   1/1     Running   0          89s
```

## 環境の確認

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
6af74b50-a213-4f9e-993b-c44052e2d03c (id: 18)
```

### デバイス

デバイスを確認する。

```sh
ip -d link show cali1e0fc2b6f3e
```

```text
30: cali1e0fc2b6f3e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 6af74b50-a213-4f9e-993b-c44052e2d03c promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 6af74b50-a213-4f9e-993b-c44052e2d03c ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: eth0@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 26:0b:35:13:15:c2 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show cali1e0fc2b6f3e
```

```text
30: cali1e0fc2b6f3e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 6af74b50-a213-4f9e-993b-c44052e2d03c
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 6af74b50-a213-4f9e-993b-c44052e2d03c ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 26:0b:35:13:15:c2 brd ff:ff:ff:ff:ff:ff link-netns 9aff30be-081c-4026-a352-fbac45faf19d
    inet 172.17.2.33/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::240b:35ff:fe13:15c2/64 scope link proto kernel_ll
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
172.17.2.33 dev cali1e0fc2b6f3e scope link
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 6af74b50-a213-4f9e-993b-c44052e2d03c ip route show
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
172.17.2.19 dev cali81f8b902e6e lladdr b6:e5:41:76:9e:08 REACHABLE
172.16.0.199 dev enp1s0 lladdr fe:54:00:49:41:b8 REACHABLE
172.17.2.17 dev calide6f5a24f1d lladdr da:2f:c2:fc:35:10 REACHABLE
172.17.2.16 dev calif35f658de0d lladdr aa:cc:4a:61:51:80 REACHABLE
172.17.2.14 dev cali30782974e56 lladdr 06:7b:37:72:06:f8 REACHABLE
172.17.2.15 dev cali41393fe08f0 lladdr 6a:a6:1d:2d:0a:b7 REACHABLE
172.17.2.22 dev cali0817bf63151 lladdr 0e:d8:ec:61:04:06 REACHABLE
172.16.0.254 dev enp1s0 lladdr 52:54:00:65:c6:68 REACHABLE
172.17.2.33 dev cali1e0fc2b6f3e lladdr 26:0b:35:13:15:c2 REACHABLE
172.17.2.10 dev cali672f36e7b7a lladdr 8a:67:37:08:6e:e1 REACHABLE
172.17.2.12 dev cali93ae5b22e23 lladdr de:47:01:a6:23:36 REACHABLE
```

ネットワーク名前空間内の ARP テーブルを確認する。

```sh
ip netns exec 6af74b50-a213-4f9e-993b-c44052e2d03c ip neigh
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
                 ct state related,established counter packets 12978863 bytes 6051082134 accept
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 59479 bytes 3576193 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-from-host-endpoint {
        }

        chain cali-POSTROUTING {
                 meta mark & 0x00010000 == 0x00010000 counter packets 179 bytes 18193 return
                 counter packets 12916192 bytes 6830047996 meta mark set mark and 0xffe4ffff
                 ct status dnat counter packets 4148278 bytes 3219327780 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority mangle; policy accept;
                 counter packets 13038342 bytes 6054658327 jump cali-PREROUTING
        }

        chain POSTROUTING {
                type filter hook postrouting priority mangle; policy accept;
                 counter packets 12916371 bytes 6830066189 jump cali-POSTROUTING
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain KUBE-FIREWALL {
                ip saddr != 127.0.0.0/8 ip daddr 127.0.0.0/8  ct status dnat counter packets 0 bytes 0 drop
        }

        chain OUTPUT {
                type filter hook output priority filter; policy accept;
                 counter packets 12910440 bytes 6829238625 jump cali-OUTPUT
                ct state new  counter packets 87429 bytes 5286216 jump KUBE-PROXY-FIREWALL
                ct state new  counter packets 87429 bytes 5286216 jump KUBE-SERVICES
                counter packets 12957456 bytes 6856425638 jump KUBE-FIREWALL
        }

        chain INPUT {
                type filter hook input priority filter; policy accept;
                 counter packets 13032389 bytes 6053829443 jump cali-INPUT
                ct state new  counter packets 59751 bytes 3585060 jump KUBE-PROXY-FIREWALL
                 counter packets 10795859 bytes 5569642955 jump KUBE-NODEPORTS
                ct state new  counter packets 59751 bytes 3585060 jump KUBE-EXTERNAL-SERVICES
                counter packets 10802586 bytes 5574784814 jump KUBE-FIREWALL
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-EXTERNAL-SERVICES {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                 counter packets 5951 bytes 828780 jump cali-FORWARD
                ct state new  counter packets 183 bytes 18565 jump KUBE-PROXY-FIREWALL
                 counter packets 187 bytes 19237 jump KUBE-FORWARD
                ct state new  counter packets 183 bytes 18565 jump KUBE-SERVICES
                ct state new  counter packets 183 bytes 18565 jump KUBE-EXTERNAL-SERVICES
                  meta mark & 0x00010000 == 0x00010000 counter packets 179 bytes 18193 accept
                 counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain KUBE-NODEPORTS {
                meta l4proto tcp  tcp dport 31115 counter packets 0 bytes 0 accept
                meta l4proto tcp  tcp dport 31115 counter packets 0 bytes 0 accept
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
                 counter packets 2276615 bytes 505489140 jump cali-from-wl-dispatch
                  counter packets 8 bytes 480 accept
        }

        chain cali-OUTPUT {
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                oifname "cali*"  counter packets 2354151 bytes 1909565649 return
                meta l4proto udp   udp dport 4789 fib saddr type local xt match "set" counter packets 0 bytes 0 accept
                 counter packets 10556289 bytes 4919672976 meta mark set mark and 0xffe4ffff
                 ct status dnat counter packets 8634284 bytes 3600082078 jump cali-to-host-endpoint
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
                iifname "cali0*"  counter packets 537 bytes 771521 goto cali-from-wl-dispatch-0
                iifname "cali1e0fc2b6f3e"  counter packets 608 bytes 68465 goto cali-fw-cali1e0fc2b6f3e
                iifname "cali30782974e56"  counter packets 226 bytes 40165 goto cali-fw-cali30782974e56
                iifname "cali41393fe08f0"  counter packets 364 bytes 29008 goto cali-fw-cali41393fe08f0
                iifname "cali53023bf1b9d"  counter packets 0 bytes 0 goto cali-fw-cali53023bf1b9d
                iifname "cali672f36e7b7a"  counter packets 572 bytes 73932 goto cali-fw-cali672f36e7b7a
                iifname "cali81f8b902e6e"  counter packets 4095 bytes 558012 goto cali-fw-cali81f8b902e6e
                iifname "cali93ae5b22e23"  counter packets 373 bytes 29767 goto cali-fw-cali93ae5b22e23
                iifname "calide6f5a24f1d"  counter packets 199 bytes 17875 goto cali-fw-calide6f5a24f1d
                iifname "calif35f658de0d"  counter packets 906 bytes 411451 goto cali-fw-calif35f658de0d
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
                 ct state related,established counter packets 281628 bytes 19129524 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 10 bytes 976 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 10 bytes 976 jump cali-pro-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 10 bytes 976 return
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
                 ct state related,established counter packets 281760 bytes 18887854 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 11 bytes 1095 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 11 bytes 1095 jump cali-pro-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 11 bytes 1095 return
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
                 ct state related,established counter packets 15 bytes 2431 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 66 bytes 7115 meta mark set mark and 0xfffcffff
                 counter packets 66 bytes 7115 jump cali-pri-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 66 bytes 7115 return
                 counter packets 0 bytes 0 jump cali-pri-_u2Tn2rSoAPffvE7JO6
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-cali30782974e56 {
                 ct state related,established counter packets 10156 bytes 1816063 accept
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
                  counter packets 21 bytes 2071 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 21 bytes 2071 log prefix "ARE0|kns.kube-system" group 2 snaplen 80
        }

        chain cali-pro-_u2Tn2rSoAPffvE7JO6 {
                  counter packets 0 bytes 0
        }

        chain cali-pro-_eY4Bnp6m80Op5FOwqd {
                  counter packets 0 bytes 0
        }

        chain cali-fw-calif35f658de0d {
                 ct state related,established counter packets 34940 bytes 18000357 accept
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
                iifname "cali*"  counter packets 2276615 bytes 505489140 goto cali-wl-to-host
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 10755774 bytes 5548340303 meta mark set mark and 0xffe4ffff
                 counter packets 10755774 bytes 5548340303 jump cali-from-host-endpoint
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
                 ct state related,established counter packets 9468 bytes 861448 accept
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
                oifname "cali0*"  counter packets 0 bytes 0 goto cali-to-wl-dispatch-0
                oifname "cali1e0fc2b6f3e"  counter packets 0 bytes 0 goto cali-tw-cali1e0fc2b6f3e
                oifname "cali30782974e56"  counter packets 0 bytes 0 goto cali-tw-cali30782974e56
                oifname "cali41393fe08f0"  counter packets 0 bytes 0 goto cali-tw-cali41393fe08f0
                oifname "cali53023bf1b9d"  counter packets 0 bytes 0 goto cali-tw-cali53023bf1b9d
                oifname "cali672f36e7b7a"  counter packets 0 bytes 0 goto cali-tw-cali672f36e7b7a
                oifname "cali81f8b902e6e"  counter packets 0 bytes 0 goto cali-tw-cali81f8b902e6e
                oifname "cali93ae5b22e23"  counter packets 0 bytes 0 goto cali-tw-cali93ae5b22e23
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
                 ct state related,established counter packets 292150 bytes 21229710 accept
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
                  counter packets 139 bytes 14982 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 139 bytes 14982 log prefix "ARI0|kns.kube-system" group 1 snaplen 80
        }

        chain cali-tw-cali93ae5b22e23 {
                 ct state related,established counter packets 13 bytes 2293 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 73 bytes 7867 meta mark set mark and 0xfffcffff
                 counter packets 73 bytes 7867 jump cali-pri-kns.kube-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 73 bytes 7867 return
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
                 counter packets 5951 bytes 828780 meta mark set mark and 0xffe5ffff
                 meta mark & 0x00010000 == 0x00000000 counter packets 5951 bytes 828780 jump cali-from-hep-forward
                iifname "cali*"  counter packets 5893 bytes 814216 jump cali-from-wl-dispatch
                oifname "cali*"  counter packets 214 bytes 30566 jump cali-to-wl-dispatch
                 counter packets 179 bytes 18193 jump cali-to-hep-forward
                 counter packets 179 bytes 18193 jump cali-cidr-block
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
                 counter packets 14 bytes 840 meta mark set mark and 0xfffcffff
                 counter packets 14 bytes 840 jump cali-pri-kns.nginx-gateway
                  meta mark & 0x00010000 == 0x00010000 counter packets 14 bytes 840 return
                 counter packets 0 bytes 0 jump cali-pri-_VZu5ATptS7dSAoXoNk
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-kns.nginx-gateway {
                  counter packets 14 bytes 840 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 14 bytes 840 log prefix "ARI0|kns.nginx-gateway" group 1 snaplen 80
        }

        chain cali-pri-_VZu5ATptS7dSAoXoNk {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali81f8b902e6e {
                 ct state related,established counter packets 690058 bytes 192769056 accept
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

        chain cali-fw-cali0817bf63151 {
                 ct state related,established counter packets 636987 bytes 229297565 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 jump cali-pro-kns.metallb-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 1 bytes 60 return
                 counter packets 0 bytes 0 jump cali-pro-_rfesb_Nv6QzsjWHy5M
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-kns.metallb-system {
                  counter packets 1 bytes 60 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 1 bytes 60 log prefix "ARE0|kns.metallb-system" group 2 snaplen 80
        }

        chain cali-pro-_rfesb_Nv6QzsjWHy5M {
                  counter packets 0 bytes 0
        }

        chain cali-to-wl-dispatch-0 {
                oifname "cali0817bf63151"  counter packets 0 bytes 0 goto cali-tw-cali0817bf63151
                oifname "cali0cd00cf4c0f"  counter packets 0 bytes 0 goto cali-tw-cali0cd00cf4c0f
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-kns.metallb-system {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARI0|kns.metallb-system" group 1 snaplen 80
        }

        chain cali-pri-_rfesb_Nv6QzsjWHy5M {
                  counter packets 0 bytes 0
        }

        chain cali-from-wl-dispatch-0 {
                iifname "cali0817bf63151"  counter packets 636988 bytes 229297625 goto cali-fw-cali0817bf63151
                iifname "cali0cd00cf4c0f"  counter packets 0 bytes 0 goto cali-fw-cali0cd00cf4c0f
                  counter packets 0 bytes 0 drop
        }

        chain cali-tw-cali0817bf63151 {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.metallb-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_rfesb_Nv6QzsjWHy5M
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-fw-cali1e0fc2b6f3e {
                 ct state related,established counter packets 29244 bytes 2039715 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 jump cali-pro-kns.ingress-nginx
                  meta mark & 0x00010000 == 0x00010000 counter packets 1 bytes 60 return
                 counter packets 0 bytes 0 jump cali-pro-_WuAV8wMhwxuQO3vuFE
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRE" group 2 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-_WuAV8wMhwxuQO3vuFE {
                  counter packets 0 bytes 0
        }

        chain cali-tw-cali1e0fc2b6f3e {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.ingress-nginx
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_WuAV8wMhwxuQO3vuFE
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 log prefix "DRI" group 1 snaplen 80
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-_WuAV8wMhwxuQO3vuFE {
                  counter packets 0 bytes 0
        }

        chain cali-pro-kns.ingress-nginx {
                  counter packets 1 bytes 60 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 1 bytes 60 log prefix "ARE0|kns.ingress-nginx" group 2 snaplen 80
        }

        chain cali-pri-kns.ingress-nginx {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 log prefix "ARI0|kns.ingress-nginx" group 1 snaplen 80
        }
}
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-SERVICES {
                meta l4proto tcp ip daddr 10.99.161.198  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-GZ25SP4UFGF7SAVL
                meta l4proto tcp ip daddr 10.96.0.1  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-NPX46M4PTMTKRN6Y
                meta l4proto udp ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-TCOU7JCQXEZGVUNU
                meta l4proto tcp ip daddr 10.111.96.199  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-I24EZXP75AX5E7TU
                meta l4proto tcp ip daddr 10.107.225.207  tcp dport 7443 counter packets 0 bytes 0 jump KUBE-SVC-RMIM2MJTF3UE534N
                meta l4proto tcp ip daddr 10.98.161.214  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-SVC-RK657RLKDNVNU64O
                meta l4proto tcp ip daddr 10.104.39.6  tcp dport 8081 counter packets 0 bytes 0 jump KUBE-SVC-UA7HCJMIMBJJDU4H
                meta l4proto tcp ip daddr 10.105.48.93  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-SEU3WZYSL6OBXFRO
                meta l4proto tcp ip daddr 10.105.223.177  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                meta l4proto tcp ip daddr 172.16.0.102  tcp dport 443 counter packets 0 bytes 0 jump KUBE-EXT-EDNDUDH2C75GIR6O
                meta l4proto tcp ip daddr 10.105.159.37  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-EZYNCFY2F7N6OQA2
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-ERIFXISQEP7F7OF4
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-SVC-JD5MR3NA4I4DYORP
                meta l4proto tcp ip daddr 10.105.223.177  tcp dport 80 counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                meta l4proto tcp ip daddr 172.16.0.102  tcp dport 80 counter packets 0 bytes 0 jump KUBE-EXT-CG5I4G2RS3ZVWGLK
                 fib daddr type local counter packets 1225 bytes 73500 jump KUBE-NODEPORTS
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                 counter packets 84978 bytes 5124200 jump cali-OUTPUT
                 counter packets 85561 bytes 5162894 jump KUBE-SERVICES
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                 counter packets 189 bytes 18793 jump cali-PREROUTING
                 counter packets 203 bytes 19765 jump KUBE-SERVICES
        }

        chain KUBE-POSTROUTING {
                meta mark & 0x00004000 != 0x00004000 counter packets 1861 bytes 111891 return
                counter packets 0 bytes 0 meta mark set mark xor 0x4000
                 counter packets 0 bytes 0 masquerade fully-random
        }

        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                 counter packets 85646 bytes 5172923 jump KUBE-POSTROUTING
                 counter packets 85028 bytes 5133261 jump cali-POSTROUTING
        }

        chain KUBE-NODEPORTS {
                meta l4proto tcp  tcp dport 32727 counter packets 0 bytes 0 jump KUBE-EXT-EDNDUDH2C75GIR6O
                meta l4proto tcp  tcp dport 32476 counter packets 0 bytes 0 jump KUBE-EXT-CG5I4G2RS3ZVWGLK
        }

        chain KUBE-MARK-MASQ {
                counter packets 0 bytes 0 meta mark set mark or 0x4000
        }

        chain KUBE-SVC-NPX46M4PTMTKRN6Y {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.1  tcp dport 443 counter packets 16 bytes 960 jump KUBE-MARK-MASQ
                 counter packets 16 bytes 960 jump KUBE-SEP-23Y66C2VAJ3WDEMI
        }

        chain KUBE-SEP-23Y66C2VAJ3WDEMI {
                ip saddr 172.16.0.11  counter packets 16 bytes 960 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 16 bytes 960 dnat to 172.16.0.11:6443
        }

        chain KUBE-SVC-TCOU7JCQXEZGVUNU {
                meta l4proto udp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-5FL5ISI2YRT6Y7BH
                 counter packets 0 bytes 0 jump KUBE-SEP-UHAP4MO6KX56B2GW
        }

        chain KUBE-SEP-5FL5ISI2YRT6Y7BH {
                ip saddr 172.17.2.12  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp  meta l4proto udp counter packets 0 bytes 0 dnat to 172.17.2.12:53
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
                meta l4proto udp  meta l4proto udp counter packets 0 bytes 0 dnat to 172.17.2.15:53
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
                 counter packets 189 bytes 18793 jump cali-fip-dnat
        }

        chain cali-fip-dnat {
        }

        chain cali-POSTROUTING {
                 counter packets 85028 bytes 5133261 jump cali-fip-snat
                 counter packets 85028 bytes 5133261 jump cali-nat-outgoing
                oifname "vxlan.calico"  fib saddr . oif type != local fib saddr type local counter packets 0 bytes 0 masquerade fully-random
        }

        chain cali-fip-snat {
        }

        chain cali-nat-outgoing {
                 xt match "set" xt match "set" counter packets 23 bytes 2191 masquerade fully-random
        }

        chain cali-OUTPUT {
                 counter packets 84978 bytes 5124200 jump cali-fip-dnat
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

        chain KUBE-SVC-GZ25SP4UFGF7SAVL {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.99.161.198  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-FHPKWWKTQLUCRSSK
        }

        chain KUBE-SEP-FHPKWWKTQLUCRSSK {
                ip saddr 172.17.2.22  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.22:9443
        }

        chain KUBE-EXT-EDNDUDH2C75GIR6O {
                ip saddr 172.17.0.0/16  counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                counter packets 0 bytes 0 jump KUBE-SVL-EDNDUDH2C75GIR6O
        }

        chain KUBE-SVC-EDNDUDH2C75GIR6O {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.105.223.177  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-K6ZN3TYLR4BB47UR
        }

        chain KUBE-SVL-EDNDUDH2C75GIR6O {
                 counter packets 0 bytes 0 jump KUBE-SEP-K6ZN3TYLR4BB47UR
        }

        chain KUBE-SEP-K6ZN3TYLR4BB47UR {
                ip saddr 172.17.2.33  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.33:443
        }

        chain KUBE-SVC-EZYNCFY2F7N6OQA2 {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.105.159.37  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-A4G7DTJZT3RGZA5W
        }

        chain KUBE-SEP-A4G7DTJZT3RGZA5W {
                ip saddr 172.17.2.33  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.33:8443
        }

        chain KUBE-EXT-CG5I4G2RS3ZVWGLK {
                ip saddr 172.17.0.0/16  counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                counter packets 0 bytes 0 jump KUBE-SVL-CG5I4G2RS3ZVWGLK
        }

        chain KUBE-SVC-CG5I4G2RS3ZVWGLK {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.105.223.177  tcp dport 80 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-GJW35FPJJQYZOHKJ
        }

        chain KUBE-SVL-CG5I4G2RS3ZVWGLK {
                 counter packets 0 bytes 0 jump KUBE-SEP-GJW35FPJJQYZOHKJ
        }

        chain KUBE-SEP-GJW35FPJJQYZOHKJ {
                ip saddr 172.17.2.33  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 172.17.2.33:80
        }
}
# Warning: table ip raw is managed by iptables-nft, do not touch!
table ip raw {
        chain cali-PREROUTING {
                 counter packets 13038342 bytes 6054658327 meta mark set mark and 0xffe4ffff
                meta l4proto udp  udp dport 4789 counter packets 0 bytes 0 notrack
                iifname "cali*"  counter packets 2282508 bytes 506303356 meta mark set mark or 0x80000
                 meta mark & 0x00080000 == 0x00080000 counter packets 2282508 bytes 506303356 jump cali-rpf-skip
                 meta mark & 0x00080000 == 0x00080000 fib saddr . mark . iif oif 0 counter packets 0 bytes 0 drop
                 meta mark & 0x00080000 == 0x00000000 counter packets 10755834 bytes 5548354971 jump cali-from-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-rpf-skip {
        }

        chain cali-from-host-endpoint {
        }

        chain cali-OUTPUT {
                 counter packets 12910442 bytes 6829238729 meta mark set mark and 0xffe4ffff
                 counter packets 12910442 bytes 6829238729 jump cali-to-host-endpoint
                meta l4proto udp  udp dport 4789 counter packets 0 bytes 0 notrack
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority raw; policy accept;
                 counter packets 13038342 bytes 6054658327 jump cali-PREROUTING
        }

        chain OUTPUT {
                type filter hook output priority raw; policy accept;
                 counter packets 12910442 bytes 6829238729 jump cali-OUTPUT
        }
}
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n ingress-nginx -o wide
```

```text
NAME                                           READY   STATUS    RESTARTS   AGE     IP            NODE                    NOMINATED NODE   READINESS GATES
pod/ingress-nginx-controller-7f894db6f-kcb92   1/1     Running   0          4m33s   172.17.2.33   controller.home.local   <none>           <none>

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE     SELECTOR
service/ingress-nginx-controller             LoadBalancer   10.105.223.177   172.16.0.102   80:32476/TCP,443:32727/TCP   4m34s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
service/ingress-nginx-controller-admission   ClusterIP      10.105.159.37    <none>         443/TCP                      4m34s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                                                                                                                     SELECTOR
deployment.apps/ingress-nginx-controller   1/1     1            1           4m34s   controller   registry.k8s.io/ingress-nginx/controller:v1.13.3@sha256:1b044f6dcac3afbb59e05d98463f1dec6f3d3fb99940bc12ca5d80270358e3bd   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx

NAME                                                 DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                                                                                                                     SELECTOR
replicaset.apps/ingress-nginx-controller-7f894db6f   1         1         1       4m34s   controller   registry.k8s.io/ingress-nginx/controller:v1.13.3@sha256:1b044f6dcac3afbb59e05d98463f1dec6f3d3fb99940bc12ca5d80270358e3bd   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,pod-template-hash=7f894db6f
```

```sh
kubectl describe service ingress-nginx-controller -n ingress-nginx
```

```text
Name:                     ingress-nginx-controller
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=ingress-nginx
                          app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
                          app.kubernetes.io/version=1.13.3
Annotations:              metallb.io/ip-allocated-from-pool: public-pool
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.105.223.177
IPs:                      10.105.223.177
LoadBalancer Ingress:     172.16.0.102 (VIP)
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  32476/TCP
Endpoints:                172.17.2.33:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  32727/TCP
Endpoints:                172.17.2.33:443
Session Affinity:         None
External Traffic Policy:  Local
Internal Traffic Policy:  Cluster
HealthCheck NodePort:     31115
Events:
  Type    Reason       Age   From                Message
  ----    ------       ----  ----                -------
  Normal  IPAllocated  5m1s  metallb-controller  Assigned IP ["172.16.0.102"]
```

```sh
kubectl describe service ingress-nginx-controller-admission -n ingress-nginx
```

```text
Name:                     ingress-nginx-controller-admission
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=ingress-nginx
                          app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
                          app.kubernetes.io/version=1.13.3
Annotations:              <none>
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.105.159.37
IPs:                      10.105.159.37
Port:                     https-webhook  443/TCP
TargetPort:               webhook/TCP
Endpoints:                172.17.2.33:8443
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### コンテナ

ワーカノードで確認する。

```sh
crictl ps -a
```

```text
CONTAINER           IMAGE                                                                                                              CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                             NAMESPACE
3a9bb81bff18f       registry.k8s.io/ingress-nginx/controller@sha256:7b4073fc95e078d863c0b0b08deb72e01d2cf629e2156822bcd394fc2bcd8e83   18 minutes ago      Running             controller                  0                   2a31814eff6c5       ingress-nginx-controller-7f894db6f-kcb92        ingress-nginx
6a9e197a4b8a5       quay.io/metallb/speaker@sha256:7dbe5e4b1ee3be1638b1620e2670649e9900768972060a31d2e809855e31e344                    2 hours ago         Running             speaker                     0                   d04db24e8f2fe       speaker-n7prq                                   metallb-system
7f31331627784       quay.io/metallb/controller@sha256:9bd30b4ee165ad6bababa6334594d854b8c1037c385af83832fb07a4b2c23212                 2 hours ago         Running             controller                  0                   88e1eacb0618f       controller-58fdf44d87-tgp79                     metallb-system
35104e7c5aadc       44c116ec3db7fbb99aafe27302134b6de6cced67a71c7d8fd7d7dfa77cfc242f                                                   3 hours ago         Running             nginx-gateway               0                   1e132d225a3c2       nginx-gateway-6477c87c8-xq9vv                   nginx-gateway
7e86e0b89f60e       a7d029fd8f6be94c26af980675c1650818e1e6e19dbd2f8c13e6e61963f021e8                                                   5 hours ago         Running             goldmane                    2                   b6e6d502bdf37       goldmane-85c8f6d476-qckwz                       calico-system
a4efeced770df       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a                                                   5 hours ago         Running             calico-apiserver            2                   b9a9f507e4213       calico-apiserver-5c49f7b8cd-5thsd               calico-apiserver
4be35211a40b9       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b                                                   5 hours ago         Running             coredns                     2                   ed7589b8ee54a       coredns-674b8bbfcf-csznq                        kube-system
fa2c52c47e221       879f2443aed0573271114108bfec35d3e76419f98282ef796c646d0986c5ba6a                                                   5 hours ago         Running             calico-apiserver            2                   2303f2c3ebed6       calico-apiserver-5c49f7b8cd-cw8rw               calico-apiserver
2c32eaa41ddbb       7e29b0984d517678aab6ca138482c318989f6f28daf9d3b5dd6e4a5a3115ac16                                                   5 hours ago         Running             whisker-backend             2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
dabfc11320849       9a4eedeed4a531acefb7f5d0a1b7e3856b1a9a24d9e7d25deef2134d7a734c2d                                                   5 hours ago         Running             whisker                     2                   99c18ede38b56       whisker-5cd776c89c-tcqwl                        calico-system
427dc8955c1dc       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b                                                   5 hours ago         Running             coredns                     2                   5c91d08fb137b       coredns-674b8bbfcf-ng4p7                        kube-system
c883af54c988d       b8f31c4fdaed3fa08af64de3d37d65a4c2ea0d9f6f522cb60d2e0cb424f8dd8a                                                   5 hours ago         Running             csi-node-driver-registrar   2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
c99ee13428df9       666f4e02e75c30547109a06ed75b415a990a970811173aa741379cfaac4d9dd7                                                   5 hours ago         Running             calico-csi                  2                   c72230aa478c2       csi-node-driver-7dbkx                           calico-system
8b259a4736bf6       df191a54fb79de3c693f8b1b864a1bd3bd14f63b3fff9d5fa4869c471ce3cd37                                                   5 hours ago         Running             calico-kube-controllers     2                   6fb73445c5b3a       calico-kube-controllers-98fbcc856-vgxhq         calico-system
b0bb7e3a28a19       ce9c4ac0f175f22c56e80844e65379d9ebe1d8a4e2bbb38dc1db0f53a8826f0f                                                   5 hours ago         Running             calico-node                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
2d63dd1aa09a5       034822460c2f667e1f4a7679c843cc35ce1bf2c25dec86f04e07fb403df7e458                                                   5 hours ago         Exited              install-cni                 2                   401470c9b90cf       calico-node-7lw6s                               calico-system
dbbc87a2e095e       4f2b088ed6fdfc6a97ac0650a4ba8171107d6656ce265c592e4c8423fd10e5c4                                                   5 hours ago         Exited              flexvol-driver              2                   401470c9b90cf       calico-node-7lw6s                               calico-system
5c60b077e08fd       1911afdd8478c6ca3036ff85614050d5d19acc0f0c3f6a5a7b3e34b38dd309c9                                                   5 hours ago         Running             tigera-operator             2                   b00e4ea97c8cd       tigera-operator-755d956888-2l5ks                tigera-operator
ce2a879a71387       2844ee7bb56c2c194e1f4adafb9e7b60b9ed16aa4d07ab8ad1f019362e2efab3                                                   5 hours ago         Running             kube-proxy                  2                   862ab4aed1086       kube-proxy-xn5ft                                kube-system
55ad4fde21a9f       1d7bb7b0cce2924d35c7c26f6b6600409ea7c9535074c3d2e517ffbb3a0e0b36                                                   5 hours ago         Running             calico-typha                2                   4d2521ead8ea9       calico-typha-6574bcdfc5-fgdgz                   calico-system
c64f172909e7e       33b680aadf474b7e5e73957fc00c6af86dd0484c699c8461ba33ee656d1823bf                                                   5 hours ago         Running             kube-scheduler              3                   0246fe2061ab7       kube-scheduler-controller.home.local            kube-system
4d4ad99f7cadb       8bb43160a0df4d7d34c89d9edbc48735bc2f830771e4b501937338221be0f668                                                   5 hours ago         Running             kube-controller-manager     3                   661ffd4ff1052       kube-controller-manager-controller.home.local   kube-system
886b87e7b5891       b7335a56022aba291f5df653c01b7ab98d64fb5cab221378617f4a1236e06a62                                                   5 hours ago         Running             kube-apiserver              3                   f17ac77d88354       kube-apiserver-controller.home.local            kube-system
c75beff2be6a1       499038711c0816eda03a1ad96a8eb0440c005baa6949698223c6176b7f5077e1                                                   5 hours ago         Running             etcd                        3                   0b701b25c64e8       etcd-controller.home.local                      kube-system
```

controller が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 3a9bb81bff18f | jq '.info.pid')
```

```text
6af74b50-a213-4f9e-993b-c44052e2d03c
```

コンテナイメージを確認する。

```sh
crictl images
```

```text
IMAGE                                                TAG                 IMAGE ID            SIZE
docker.io/calico/apiserver                           v3.30.3             879f2443aed05       118MB
docker.io/calico/cni                                 v3.30.3             034822460c2f6       162MB
docker.io/calico/csi                                 v3.30.3             666f4e02e75c3       20.6MB
docker.io/calico/goldmane                            v3.30.3             a7d029fd8f6be       151MB
docker.io/calico/kube-controllers                    v3.30.3             df191a54fb79d       122MB
docker.io/calico/node-driver-registrar               v3.30.3             b8f31c4fdaed3       33.3MB
docker.io/calico/node                                v3.30.3             ce9c4ac0f175f       403MB
docker.io/calico/pod2daemon-flexvol                  v3.30.3             4f2b088ed6fdf       12.1MB
docker.io/calico/typha                               v3.30.3             1d7bb7b0cce29       85.2MB
docker.io/calico/whisker-backend                     v3.30.3             7e29b0984d517       75.5MB
docker.io/calico/whisker                             v3.30.3             9a4eedeed4a53       14.2MB
docker.io/kubernetesui/dashboard-api                 1.13.0              9e7701f8aae8a       55.5MB
docker.io/kubernetesui/dashboard-auth                1.3.0               a9de0489eb32f       49.3MB
docker.io/kubernetesui/dashboard-metrics-scraper     1.2.2               d9cbc9f4053ca       38.9MB
docker.io/kubernetesui/dashboard-web                 1.7.0               59f642f485d26       193MB
docker.io/library/httpd                              latest              2416cb32cb59e       120MB
docker.io/library/kong                               3.8                 9958b1a1a0ac3       382MB
ghcr.io/nginx/nginx-gateway-fabric/nginx             2.1.4               99119d5762c69       155MB
ghcr.io/nginx/nginx-gateway-fabric                   2.1.4               44c116ec3db7f       50.8MB
quay.io/metallb/controller                           v0.15.2             1b258f39a2a12       78.3MB
quay.io/metallb/speaker                              v0.15.2             1eeb60bd05864       140MB
quay.io/tigera/operator                              v1.38.6             1911afdd8478c       87.2MB
registry.k8s.io/coredns/coredns                      v1.12.0             1cf5f116067c6       71.2MB
registry.k8s.io/etcd                                 3.5.21-0            499038711c081       154MB
registry.k8s.io/ingress-nginx/controller             <none>              c44d76c3213ea       334MB
registry.k8s.io/ingress-nginx/kube-webhook-certgen   <none>              08cfe302feafe       43.1MB
registry.k8s.io/kube-apiserver                       v1.33.5             b7335a56022ab       103MB
registry.k8s.io/kube-controller-manager              v1.33.5             8bb43160a0df4       95.8MB
registry.k8s.io/kube-proxy                           v1.33.5             2844ee7bb56c2       99.3MB
registry.k8s.io/kube-scheduler                       v1.33.5             33b680aadf474       74.6MB
registry.k8s.io/pause                                3.10                873ed75102791       742kB
```

## 動作確認

ポッドを作成する。

```sh
kubectl create deployment demo-ingress --image=httpd --port=80
```

```text
deployment.apps/demo-ingress created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```text
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
demo-ingress-69f685c475-xcfph   1/1     Running   0          11s   172.17.2.34   controller.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo-ingress
```

```text
service/demo-ingress exposed
```

サービスを確認する。

```sh
kubectl get service
```

```text
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
demo-ingress   ClusterIP   10.96.143.206   <none>        80/TCP    14s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP   5h23m
```

ingress を作成する。

```sh
kubectl create ingress demo --class=nginx --rule="demo-ingress.localdev.me/*=demo-ingress:80"
```

```text
ingress.networking.k8s.io/demo created
```

ingress を確認する。

```sh
kubectl get ingress
```

```text
NAME   CLASS   HOSTS                      ADDRESS   PORTS   AGE
demo   nginx   demo-ingress.localdev.me             80      16s
```

フォワーディングする。

```sh
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

接続確認する。

```sh
curl --resolve demo-ingress.localdev.me:8080:127.0.0.1 http://demo-ingress.localdev.me:8080
```

```text
<html><body><h1>It works!</h1></body></html>
```
