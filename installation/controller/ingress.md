# Ingress Controller

## インストール

Ingress NGINX Controller を構築する。

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml
```

```
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

```{warning}
2024/9/29 時点で [#8605](https://github.com/cri-o/cri-o/pull/8605) の問題があるため
*deploy.yaml* をダウンロードして `image` の指定からタグを削除する。

image: registry.k8s.io/ingress-nginx/controller:v1.11.2@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce

  ↓

image: registry.k8s.io/ingress-nginx/controller@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce

image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3

  ↓

image: registry.k8s.io/ingress-nginx/kube-webhook-certgen@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods --namespace=ingress-nginx
```

```
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-lpfzd       0/1     Completed   0          11s
ingress-nginx-admission-patch-2jw66        0/1     Completed   0          11s
ingress-nginx-controller-675fd975d-6xq8z   0/1     Running     0          11s
```

## ファイアウォール

```{warning}
ここは正しくない。
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=8443/tcp
firewall-cmd --permanent --zone=internal --add-port=8443/tcp
firewall-cmd --reload
```

ポッドが起動するノードのファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

## 環境の確認

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```
1d17a6ce-da7c-4cae-af8f-4145dcff16cd (id: 2)
482f2d0f-575e-4e27-8710-a47af2823dd1
20be9af8-e917-4ab8-a5e9-b515c3804118 (id: 0)
2abf8663-2a57-4965-9f1b-86b7e37f60e9
```

### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:cf:af:cd brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio0
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:9d:d6:e9 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio1
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether a6:71:b3:c9:b1:b2 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.a6:71:b3:c9:b1:b2 designated_root 8000.a6:71:b3:c9:b1:b2 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  248.73 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
5: vethaf1ac987@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether c2:a2:1b:1e:b3:a9 brd ff:ff:ff:ff:ff:ff link-netns 20be9af8-e917-4ab8-a5e9-b515c3804118 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.a6:71:b3:c9:b1:b2 designated_root 8000.a6:71:b3:c9:b1:b2 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:ba:9d:ce:21:bb brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    vxlan id 4096 local 10.0.0.31 dev enp2s0 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
14: cali35a78cc5f0e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 1d17a6ce-da7c-4cae-af8f-4145dcff16cd promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 1d17a6ce-da7c-4cae-af8f-4145dcff16cd ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 86:be:62:a3:3c:3b brd ff:ff:ff:ff:ff:ff link-netns 482f2d0f-575e-4e27-8710-a47af2823dd1 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:cf:af:cd brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.31/24 brd 172.16.0.255 scope global noprefixroute enp1s0
       valid_lft forever preferred_lft forever
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:9d:d6:e9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.31/24 brd 10.0.0.255 scope global noprefixroute enp2s0
       valid_lft forever preferred_lft forever
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether a6:71:b3:c9:b1:b2 brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.1/16 brd 10.85.255.255 scope global cni0
       valid_lft forever preferred_lft forever
5: vethaf1ac987@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether c2:a2:1b:1e:b3:a9 brd ff:ff:ff:ff:ff:ff link-netns 20be9af8-e917-4ab8-a5e9-b515c3804118
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 66:ba:9d:ce:21:bb brd ff:ff:ff:ff:ff:ff
    inet 172.17.255.128/32 scope global vxlan.calico
       valid_lft forever preferred_lft forever
14: cali35a78cc5f0e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 1d17a6ce-da7c-4cae-af8f-4145dcff16cd
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 1d17a6ce-da7c-4cae-af8f-4145dcff16cd ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 86:be:62:a3:3c:3b brd ff:ff:ff:ff:ff:ff link-netns 482f2d0f-575e-4e27-8710-a47af2823dd1
    inet 172.17.255.134/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::84be:62ff:fea3:3c3b/64 scope link
       valid_lft forever preferred_lft forever
```

### ルート

ルーティングを確認する。

```sh
ip route show
```

```
default via 172.16.0.254 dev enp1s0 proto static metric 100
10.0.0.0/24 dev enp2s0 proto kernel scope link src 10.0.0.31 metric 101
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev enp1s0 proto kernel scope link src 172.16.0.31 metric 100
172.17.2.0/26 via 10.0.0.11 dev enp2s0 proto 80 onlink
172.17.51.128/26 via 10.0.0.32 dev enp2s0 proto 80 onlink
blackhole 172.17.255.128/26 proto 80
172.17.255.134 dev cali35a78cc5f0e scope link
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 1d17a6ce-da7c-4cae-af8f-4145dcff16cd ip route show
```

```
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
```

### nftables

ルールセットを確認する。

```sh
nft list ruleset ip
```

```
# Warning: table ip mangle is managed by iptables-nft, do not touch!
table ip mangle {
        chain KUBE-IPTABLES-HINT {
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain cali-to-host-endpoint {
        }

        chain cali-PREROUTING {
                 ct state related,established counter packets 120818 bytes 141141380 accept
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 2100 bytes 131368 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-from-host-endpoint {
        }

        chain cali-POSTROUTING {
                 meta mark & 0x00010000 == 0x00010000 counter packets 4 bytes 240 return
                 counter packets 79128 bytes 6856878 meta mark set mark and 0xfff0ffff
                 ct status dnat counter packets 3535 bytes 1295707 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
        }

        chain PREROUTING {
                type filter hook prerouting priority mangle; policy accept;
                 counter packets 122918 bytes 141272748 jump cali-PREROUTING
        }

        chain POSTROUTING {
                type filter hook postrouting priority mangle; policy accept;
                 counter packets 79132 bytes 6857118 jump cali-POSTROUTING
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain KUBE-FIREWALL {
                ip saddr != 127.0.0.0/8 ip daddr 127.0.0.0/8  ct status dnat counter packets 0 bytes 0 drop
        }

        chain OUTPUT {
                type filter hook output priority filter; policy accept;
                 counter packets 76345 bytes 5671393 jump cali-OUTPUT
                ct state new  counter packets 5789 bytes 357522 jump KUBE-PROXY-FIREWALL
                ct state new  counter packets 5789 bytes 357522 jump KUBE-SERVICES
                counter packets 148549 bytes 9822880 jump KUBE-FIREWALL
        }

        chain INPUT {
                type filter hook input priority filter; policy accept;
                 counter packets 120130 bytes 140086963 jump cali-INPUT
                ct state new  counter packets 2096 bytes 131128 jump KUBE-PROXY-FIREWALL
                 counter packets 280565 bytes 382254359 jump KUBE-NODEPORTS
                ct state new  counter packets 2096 bytes 131128 jump KUBE-EXTERNAL-SERVICES
                counter packets 302724 bytes 414814143 jump KUBE-FIREWALL
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-EXTERNAL-SERVICES {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                 counter packets 2788 bytes 1185785 jump cali-FORWARD
                ct state new  counter packets 4 bytes 240 jump KUBE-PROXY-FIREWALL
                 counter packets 4 bytes 240 jump KUBE-FORWARD
                ct state new  counter packets 4 bytes 240 jump KUBE-SERVICES
                ct state new  counter packets 4 bytes 240 jump KUBE-EXTERNAL-SERVICES
                  meta mark & 0x00010000 == 0x00010000 counter packets 4 bytes 240 accept
                 counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain KUBE-NODEPORTS {
                meta l4proto tcp  tcp dport 31368 counter packets 0 bytes 0 accept
                meta l4proto tcp  tcp dport 31368 counter packets 0 bytes 0 accept
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

        chain cali-INPUT {
                meta l4proto udp   udp dport 4789 xt match "set" fib daddr type local counter packets 0 bytes 0 accept
                meta l4proto udp   udp dport 4789 fib daddr type local counter packets 0 bytes 0 drop
                iifname "cali*"  counter packets 1538 bytes 135792 goto cali-wl-to-host
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 118592 bytes 139951171 meta mark set mark and 0xfff0ffff
                 counter packets 118592 bytes 139951171 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-pri-_og73BH3DuNOZrbBKFW {
                  counter packets 0 bytes 0
        }

        chain cali-to-hep-forward {
        }

        chain cali-from-hep-forward {
        }

        chain cali-to-wl-dispatch {
                oifname "cali35a78cc5f0e"  counter packets 444 bytes 154866 goto cali-tw-cali35a78cc5f0e
                oifname "calia237a32da2c"  counter packets 0 bytes 0 goto cali-tw-calia237a32da2c
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-kns.calico-system {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain cali-tw-calia237a32da2c {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_og73BH3DuNOZrbBKFW
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-kns.calico-system {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain cali-FORWARD {
                 counter packets 2788 bytes 1185785 meta mark set mark and 0xfff1ffff
                 meta mark & 0x00010000 == 0x00000000 counter packets 2788 bytes 1185785 jump cali-from-hep-forward
                iifname "cali*"  counter packets 1457 bytes 279091 jump cali-from-wl-dispatch
                oifname "cali*"  counter packets 1331 bytes 906694 jump cali-to-wl-dispatch
                 counter packets 4 bytes 240 jump cali-to-hep-forward
                 counter packets 4 bytes 240 jump cali-cidr-block
        }

        chain cali-wl-to-host {
                 counter packets 1538 bytes 135792 jump cali-from-wl-dispatch
                  counter packets 0 bytes 0 accept
        }

        chain cali-OUTPUT {
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                oifname "cali*"  counter packets 1624 bytes 122634 return
                meta l4proto udp   udp dport 4789 fib saddr type local xt match "set" counter packets 0 bytes 0 accept
                 counter packets 74721 bytes 5548759 meta mark set mark and 0xfff0ffff
                 ct status dnat counter packets 73970 bytes 5438597 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-pro-_og73BH3DuNOZrbBKFW {
                  counter packets 0 bytes 0
        }

        chain cali-from-wl-dispatch {
                iifname "cali35a78cc5f0e"  counter packets 1112 bytes 157864 goto cali-fw-cali35a78cc5f0e
                iifname "calia237a32da2c"  counter packets 0 bytes 0 goto cali-fw-calia237a32da2c
                  counter packets 0 bytes 0 drop
        }

        chain cali-from-host-endpoint {
        }

        chain cali-to-host-endpoint {
        }

        chain cali-fw-calia237a32da2c {
                 ct state related,established counter packets 0 bytes 0 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 jump cali-pro-kns.calico-system
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pro-_og73BH3DuNOZrbBKFW
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                  counter packets 0 bytes 0 drop
        }

        chain cali-cidr-block {
        }

        chain cali-pro-kns.ingress-nginx {
                  counter packets 2 bytes 120 meta mark set mark or 0x10000
        }

        chain cali-pri-kns.ingress-nginx {
                  counter packets 0 bytes 0 meta mark set mark or 0x10000
        }

        chain cali-tw-cali35a78cc5f0e {
                 ct state related,established counter packets 556 bytes 408926 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 0 bytes 0 meta mark set mark and 0xfffcffff
                 counter packets 0 bytes 0 jump cali-pri-kns.ingress-nginx
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 0 bytes 0 jump cali-pri-_WuAV8wMhwxuQO3vuFE
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                  counter packets 0 bytes 0 drop
        }

        chain cali-pri-_WuAV8wMhwxuQO3vuFE {
                  counter packets 0 bytes 0
        }

        chain cali-fw-cali35a78cc5f0e {
                 ct state related,established counter packets 1242 bytes 171156 accept
                 ct state invalid counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 meta mark set mark and 0xfffcffff
                meta l4proto udp   udp dport 4789 counter packets 0 bytes 0 drop
                meta l4proto ipv4   counter packets 0 bytes 0 drop
                 counter packets 1 bytes 60 jump cali-pro-kns.ingress-nginx
                  meta mark & 0x00010000 == 0x00010000 counter packets 1 bytes 60 return
                 counter packets 0 bytes 0 jump cali-pro-_WuAV8wMhwxuQO3vuFE
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                  counter packets 0 bytes 0 drop
        }

        chain cali-pro-_WuAV8wMhwxuQO3vuFE {
                  counter packets 0 bytes 0
        }
}
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain KUBE-KUBELET-CANARY {
        }

        chain CNI-dc86429b457d5c3c87400258 {
                ip daddr 10.85.0.0/16  counter packets 0 bytes 0 accept
                ip daddr != 224.0.0.0/4  counter packets 0 bytes 0 masquerade
        }

        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                 counter packets 4979 bytes 304500 jump cali-POSTROUTING
                 counter packets 5066 bytes 310620 jump KUBE-POSTROUTING
                ip saddr 10.85.0.2  counter packets 0 bytes 0 jump CNI-dc86429b457d5c3c87400258
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-SERVICES {
                meta l4proto tcp ip daddr 10.105.183.5  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-SVC-RK657RLKDNVNU64O
                meta l4proto tcp ip daddr 10.110.103.111  tcp dport 80 counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                meta l4proto tcp ip daddr 10.110.103.111  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-ERIFXISQEP7F7OF4
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-SVC-JD5MR3NA4I4DYORP
                meta l4proto tcp ip daddr 10.96.0.1  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-NPX46M4PTMTKRN6Y
                meta l4proto tcp ip daddr 10.107.109.72  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-EZYNCFY2F7N6OQA2
                meta l4proto udp ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-TCOU7JCQXEZGVUNU
                meta l4proto tcp ip daddr 10.107.6.141  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-I24EZXP75AX5E7TU
                 fib daddr type local counter packets 354 bytes 21240 jump KUBE-NODEPORTS
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                 counter packets 4976 bytes 304320 jump cali-OUTPUT
                 counter packets 5067 bytes 310680 jump KUBE-SERVICES
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                 counter packets 1775 bytes 111868 jump cali-PREROUTING
                 counter packets 1775 bytes 111868 jump KUBE-SERVICES
        }

        chain KUBE-POSTROUTING {
                meta mark & 0x00004000 != 0x00004000 counter packets 542 bytes 32851 return
                counter packets 0 bytes 0 meta mark set mark xor 0x4000
                 counter packets 0 bytes 0 masquerade fully-random
        }

        chain KUBE-NODEPORTS {
                meta l4proto tcp  tcp dport 32157 counter packets 0 bytes 0 jump KUBE-EXT-CG5I4G2RS3ZVWGLK
                meta l4proto tcp  tcp dport 31755 counter packets 0 bytes 0 jump KUBE-EXT-EDNDUDH2C75GIR6O
        }

        chain KUBE-MARK-MASQ {
                counter packets 0 bytes 0 meta mark set mark or 0x4000
        }

        chain KUBE-SVC-ERIFXISQEP7F7OF4 {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-DCRLLDICQNJQU5TY
                 counter packets 0 bytes 0 jump KUBE-SEP-UYUAUFJDNMDWFGGG
        }

        chain KUBE-SEP-DCRLLDICQNJQU5TY {
                ip saddr 172.17.2.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.11:53
        }

        chain KUBE-SEP-UYUAUFJDNMDWFGGG {
                ip saddr 172.17.2.8  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.8:53
        }

        chain KUBE-SVC-JD5MR3NA4I4DYORP {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-EMLTEC6V2QXP7WGV
                 counter packets 0 bytes 0 jump KUBE-SEP-E47ARDV5AWIWVRT7
        }

        chain KUBE-SEP-EMLTEC6V2QXP7WGV {
                ip saddr 172.17.2.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.11:9153
        }

        chain KUBE-SEP-E47ARDV5AWIWVRT7 {
                ip saddr 172.17.2.8  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.8:9153
        }

        chain KUBE-SVC-I24EZXP75AX5E7TU {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.107.6.141  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-WUNV64UWFA5Q6SJ3
                 counter packets 0 bytes 0 jump KUBE-SEP-ONP57SP7HHS6PKBF
        }

        chain KUBE-SEP-WUNV64UWFA5Q6SJ3 {
                ip saddr 172.17.2.12  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.12:5443
        }

        chain KUBE-SEP-ONP57SP7HHS6PKBF {
                ip saddr 172.17.2.7  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.2.7:5443
        }

        chain KUBE-SVC-RK657RLKDNVNU64O {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.105.183.5  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-T7B34ACBXBTVP7YI
                 counter packets 0 bytes 0 jump KUBE-SEP-M7WXUFGNQT6WLJC2
        }

        chain KUBE-SEP-T7B34ACBXBTVP7YI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.16.0.11:5473
        }

        chain KUBE-SVC-NPX46M4PTMTKRN6Y {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.1  tcp dport 443 counter packets 31 bytes 1860 jump KUBE-MARK-MASQ
                 counter packets 35 bytes 2100 jump KUBE-SEP-23Y66C2VAJ3WDEMI
        }

        chain KUBE-SEP-23Y66C2VAJ3WDEMI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 35 bytes 2100 dnat to 172.16.0.11:6443
        }

        chain KUBE-SVC-TCOU7JCQXEZGVUNU {
                meta l4proto udp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-VK4YBHBORISVZAGF
                 counter packets 0 bytes 0 jump KUBE-SEP-5YGRFIC6ZA6ASW4V
        }

        chain KUBE-SEP-VK4YBHBORISVZAGF {
                ip saddr 172.17.2.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp   counter packets 0 bytes 0 dnat to 172.17.2.11:53
        }

        chain KUBE-SEP-5YGRFIC6ZA6ASW4V {
                ip saddr 172.17.2.8  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp   counter packets 0 bytes 0 dnat to 172.17.2.8:53
        }

        chain cali-PREROUTING {
                 counter packets 1775 bytes 111868 jump cali-fip-dnat
        }

        chain cali-fip-dnat {
        }

        chain cali-POSTROUTING {
                 counter packets 4979 bytes 304500 jump cali-fip-snat
                 counter packets 4979 bytes 304500 jump cali-nat-outgoing
                oifname "vxlan.calico"  fib saddr . oif type != local fib saddr type local counter packets 0 bytes 0 masquerade fully-random
        }

        chain cali-fip-snat {
        }

        chain cali-nat-outgoing {
                 xt match "set" xt match "set" counter packets 4 bytes 240 masquerade fully-random
        }

        chain cali-OUTPUT {
                 counter packets 4976 bytes 304320 jump cali-fip-dnat
        }

        chain KUBE-SEP-M7WXUFGNQT6WLJC2 {
                ip saddr 172.16.0.32  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.16.0.32:5473
        }

        chain KUBE-SVC-EZYNCFY2F7N6OQA2 {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.107.109.72  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-OGJBLRZDJ7AZQUIS
        }

        chain KUBE-SEP-OGJBLRZDJ7AZQUIS {
                ip saddr 172.17.255.134  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.255.134:8443
        }

        chain KUBE-EXT-CG5I4G2RS3ZVWGLK {
                ip saddr 172.17.0.0/16  counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-SVC-CG5I4G2RS3ZVWGLK
                counter packets 0 bytes 0 jump KUBE-SVL-CG5I4G2RS3ZVWGLK
        }

        chain KUBE-SVC-CG5I4G2RS3ZVWGLK {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.110.103.111  tcp dport 80 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-J6AG7Y46UJXOKODM
        }

        chain KUBE-SVL-CG5I4G2RS3ZVWGLK {
                 counter packets 0 bytes 0 jump KUBE-SEP-J6AG7Y46UJXOKODM
        }

        chain KUBE-SEP-J6AG7Y46UJXOKODM {
                ip saddr 172.17.255.134  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.255.134:80
        }

        chain KUBE-EXT-EDNDUDH2C75GIR6O {
                ip saddr 172.17.0.0/16  counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 fib saddr type local counter packets 0 bytes 0 jump KUBE-SVC-EDNDUDH2C75GIR6O
                counter packets 0 bytes 0 jump KUBE-SVL-EDNDUDH2C75GIR6O
        }

        chain KUBE-SVC-EDNDUDH2C75GIR6O {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.110.103.111  tcp dport 443 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 counter packets 0 bytes 0 jump KUBE-SEP-PI7EB3IHGGPMU3WU
        }

        chain KUBE-SVL-EDNDUDH2C75GIR6O {
                 counter packets 0 bytes 0 jump KUBE-SEP-PI7EB3IHGGPMU3WU
        }

        chain KUBE-SEP-PI7EB3IHGGPMU3WU {
                ip saddr 172.17.255.134  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.17.255.134:443
        }
}
# Warning: table ip raw is managed by iptables-nft, do not touch!
table ip raw {
        chain cali-PREROUTING {
                 counter packets 122918 bytes 141272748 meta mark set mark and 0xfff0ffff
                iifname "cali*"  counter packets 2995 bytes 414883 meta mark set mark or 0x40000
                 meta mark & 0x00040000 == 0x00040000 counter packets 2995 bytes 414883 jump cali-rpf-skip
                 meta mark & 0x00040000 == 0x00040000 fib saddr . mark . iif oif 0 counter packets 0 bytes 0 drop
                 meta mark & 0x00040000 == 0x00000000 counter packets 119923 bytes 140857865 jump cali-from-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-rpf-skip {
        }

        chain cali-from-host-endpoint {
        }

        chain cali-OUTPUT {
                 counter packets 76345 bytes 5671393 meta mark set mark and 0xfff0ffff
                 counter packets 76345 bytes 5671393 jump cali-to-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority raw; policy accept;
                 counter packets 122918 bytes 141272748 jump cali-PREROUTING
        }

        chain OUTPUT {
                type filter hook output priority raw; policy accept;
                 counter packets 76345 bytes 5671393 jump cali-OUTPUT
        }
}
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all -n ingress-nginx -o wide
```

```
NAME                                           READY   STATUS      RESTARTS   AGE    IP               NODE                  NOMINATED NODE   READINESS GATES
pod/ingress-nginx-admission-create-lpfzd       0/1     Completed   0          112s   172.17.255.133   worker01.home.local   <none>           <none>
pod/ingress-nginx-admission-patch-2jw66        0/1     Completed   0          112s   172.17.51.133    worker02.home.local   <none>           <none>
pod/ingress-nginx-controller-675fd975d-6xq8z   1/1     Running     0          112s   172.17.255.134   worker01.home.local   <none>           <none>

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE    SELECTOR
service/ingress-nginx-controller             LoadBalancer   10.110.103.111   <pending>     80:32157/TCP,443:31755/TCP   112s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
service/ingress-nginx-controller-admission   ClusterIP      10.107.109.72    <none>        443/TCP                      112s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES                                                                                                             SELECTOR
deployment.apps/ingress-nginx-controller   1/1     1            1           112s   controller   registry.k8s.io/ingress-nginx/controller@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx

NAME                                                 DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES                                                                                                             SELECTOR
replicaset.apps/ingress-nginx-controller-675fd975d   1         1         1       112s   controller   registry.k8s.io/ingress-nginx/controller@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,pod-template-hash=675fd975d

NAME                                       STATUS     COMPLETIONS   DURATION   AGE    CONTAINERS   IMAGES                                                                                                                       SELECTOR
job.batch/ingress-nginx-admission-create   Complete   1/1           4s         112s   create       registry.k8s.io/ingress-nginx/kube-webhook-certgen@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3   batch.kubernetes.io/controller-uid=281dbc91-b6b9-4d30-bbc8-d025eaefb6b3
job.batch/ingress-nginx-admission-patch    Complete   1/1           3s         112s   patch        registry.k8s.io/ingress-nginx/kube-webhook-certgen@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3   batch.kubernetes.io/controller-uid=19758c72-f6f2-4df1-8fc2-8976853a7b46
```

```sh
kubectl describe service ingress-nginx-controller -n ingress-nginx
```

```
Name:                     ingress-nginx-controller
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=ingress-nginx
                          app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
                          app.kubernetes.io/version=1.11.2
Annotations:              <none>
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.110.103.111
IPs:                      10.110.103.111
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  32157/TCP
Endpoints:                172.17.255.134:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  31755/TCP
Endpoints:                172.17.255.134:443
Session Affinity:         None
External Traffic Policy:  Local
Internal Traffic Policy:  Cluster
HealthCheck NodePort:     31368
Events:                   <none>
```

```sh
kubectl describe service ingress-nginx-controller-admission -n ingress-nginx
```

```
Name:                     ingress-nginx-controller-admission
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=ingress-nginx
                          app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
                          app.kubernetes.io/version=1.11.2
Annotations:              <none>
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.107.109.72
IPs:                      10.107.109.72
Port:                     https-webhook  443/TCP
TargetPort:               webhook/TCP
Endpoints:                172.17.255.134:8443
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

### コンテナ

ワーカノードで確認する。

```sh
crictl ps
```

```
CONTAINER           IMAGE                                                                                                            CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
681fa7bf5587c       a80c8fd6e52292d38d4e58453f310d612da59d802a3b62f4b88a21c50178f7ab                                                 4 minutes ago       Running             controller                  0                   f657d3658c6c9       ingress-nginx-controller-675fd975d-6xq8z
1e09f4362c597       docker.io/calico/node@sha256:3c0e24adecc39e89780e807400def972deb0ec9de9fbbbaceade072a8c6ae94f                    2 hours ago         Running             calico-node                 0                   2bbadeecd8d63       calico-node-n84qn
5e00e077f45c2       docker.io/calico/node-driver-registrar@sha256:f44b460f2684d33042970a7703c2b3bfa0c026980528e3b582833baf15384486   2 hours ago         Running             csi-node-driver-registrar   0                   5dac55736598c       csi-node-driver-rzkl5
92c3b3a4a6aea       docker.io/calico/csi@sha256:8375b443b2b51f04eed6d29e889bc4ce7e30d9d88a26608579de22d691473a3f                     2 hours ago         Running             calico-csi                  0                   5dac55736598c       csi-node-driver-rzkl5
cffa497347532       registry.k8s.io/kube-proxy@sha256:c31eb37ccda83c6c8ee2ee5030a7038b04ecaa393d14cb71f01ab18147366fbf               2 hours ago         Running             kube-proxy                  0                   12711152b28fd       kube-proxy-77ff5
```

controller が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 681fa7bf5587c | jq '.info.pid')
```

```
1d17a6ce-da7c-4cae-af8f-4145dcff16cd
```

コンテナイメージを確認する。

```sh
crictl images
```

```
IMAGE                                                TAG                 IMAGE ID            SIZE
docker.io/calico/cni                                 v3.28.0             107014d9f4c89       209MB
docker.io/calico/csi                                 v3.28.0             1a094aeaf1521       18.3MB
docker.io/calico/node-driver-registrar               v3.28.0             0f80feca743f4       23.5MB
docker.io/calico/node                                v3.28.0             4e42b6f329bc1       355MB
docker.io/calico/pod2daemon-flexvol                  v3.28.0             587b28ecfc62e       13.4MB
registry.k8s.io/ingress-nginx/controller             <none>              a80c8fd6e5229       289MB
registry.k8s.io/ingress-nginx/kube-webhook-certgen   <none>              ce263a8653f9c       55.8MB
registry.k8s.io/kube-proxy                           v1.31.0             ad83b2ca7b09e       92.7MB
registry.k8s.io/pause                                3.10                873ed75102791       742kB
```

## 動作確認

ポッドを作成する。

```sh
kubectl create deployment demo --image=httpd --port=80
```

```
deployment.apps/demo created
```

ポッドを確認する。

```sh
kubectl get pod -o wide
```

```
NAME                    READY   STATUS    RESTARTS   AGE   IP              NODE                  NOMINATED NODE   READINESS GATES
demo-6c87cdc7bf-57f84   1/1     Running   0          34s   172.17.51.134   worker02.home.local   <none>           <none>
```

サービスを作成する。

```sh
kubectl expose deployment demo
```

```
service/demo exposed
```

サービスを確認する。

```sh
kubectl get service
```

```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
demo         ClusterIP   10.104.7.189   <none>        80/TCP    39s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   21h
```

ingress を作成する。

```sh
kubectl create ingress demo-localhost --class=nginx --rule="demo.localdev.me/*=demo:80"
```

```{warning}
下記のエラーが発生するため firewalld を無効化した。

error: failed to create ingress: Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.107.109.72:443: connect: no route to host
```

```
ingress.networking.k8s.io/demo-localhost created
```

ingress を確認する。

```sh
kubectl get ingress
```

```
NAME             CLASS   HOSTS              ADDRESS   PORTS   AGE
demo-localhost   nginx   demo.localdev.me             80      100s
```

フォワーディングする。

```sh
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

```
Forwarding from 127.0.0.1:8080 -> 80
```

接続確認する。

```sh
curl --resolve demo.localdev.me:8080:127.0.0.1 http://demo.localdev.me:8080
```

```
<html><body><h1>It works!</h1></body></html>
```
