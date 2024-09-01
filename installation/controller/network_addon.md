# ネットワークアドオン

## Calico

### ファイアウォール

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=179/tcp
firewall-cmd --permanent --zone=public --add-port=4789/udp
firewall-cmd --permanent --zone=public --add-port=5473/tcp
firewall-cmd --permanent --zone=public --add-port=51820-51821/udp
firewall-cmd --reload
```

vxlan は eth1 がローカル IP アドレスになるので internal を開ける。

```{note}
どこで決まる？
```

```sh
firewall-cmd --permanent --zone=internal --add-port=4789/udp
firewall-cmd --reload
```

### NetworkManager

Calico が制御するデバイスは NetworkManager の制御対象から外す。

```sh
cat > /etc/NetworkManager/conf.d/calico.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
EOF
```

### インストール

Calico の Operator を構築する。

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
```

```
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
```

Calico を構築する。

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml
```

```
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
```

すべての Pod が起動するまで待つ。

```sh
watch kubectl get pods -n calico-system
```

```
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-597855f5d-pvnh4   1/1     Running   0          3m36s
calico-node-6bn67                         1/1     Running   0          3m12s
calico-typha-6655bb4b48-xqtpn             1/1     Running   0          3m40s
csi-node-driver-dpdzx                     2/2     Running   0          3m37s
```

### 環境確認

#### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```
21d70161-a16f-4d54-be99-e88c83e06a79 (id: 5)
1631d05f-a264-4956-8bd0-9851884eb00b (id: 4)
a6630dc0-bda5-4a80-8e8b-f9ed6866da34
88ddfd64-54be-4295-97e6-4a1bac51ffeb (id: 3)
d9772ef6-bc47-474a-bfcc-3a2f17bad12f (id: 2)
8658f77b-818b-48d4-8a72-a66fc57b5cff
78f5b298-c711-4967-81e5-bc94c87cb228
e3e2e0a1-062b-480a-a483-660254e2abe0 (id: 0)
c1c0eace-08c5-4bdd-9420-6b215f3fefc5
cb0e0d0a-ff37-4145-9b11-aca82942493d (id: 1)
a99f9b7e-5f7c-4ec8-bf45-cb64d51456e0
e7b1d7bb-4b7d-4ea6-8afd-8b24ee745b24
322f5081-3c51-4541-a1dc-5ac30daa5713
dfb4ea86-a48b-4ee0-bf68-62892c0f3838
```

#### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:60 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 0ad645ee-5652-4cb9-8a61-d70dbad17ffe
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:63 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev e64b23d6-2347-4eb9-9ab7-3274451a7178
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2e:ef:76:4d:74:2f brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer   88.97 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
5: veth2c3744a3@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 76:c7:d5:bb:81:33 brd ff:ff:ff:ff:ff:ff link-netns e3e2e0a1-062b-480a-a483-660254e2abe0 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
6: vethf3cf048b@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 4a:9e:48:be:23:c9 brd ff:ff:ff:ff:ff:ff link-netns cb0e0d0a-ff37-4145-9b11-aca82942493d promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
7: veth1d625a4b@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 9e:09:10:dd:b5:13 brd ff:ff:ff:ff:ff:ff link-netns d9772ef6-bc47-474a-bfcc-3a2f17bad12f promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8003 port_no 0x3 designated_port 32771 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
8: veth0593223e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 76:52:17:7f:a4:02 brd ff:ff:ff:ff:ff:ff link-netns 88ddfd64-54be-4295-97e6-4a1bac51ffeb promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8004 port_no 0x4 designated_port 32772 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
11: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:b9:7e:86:1e:53 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    vxlan id 4096 local 10.0.0.11 dev eth1 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
12: cali9d04617f188@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 1631d05f-a264-4956-8bd0-9851884eb00b promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
13: cali1dac8782bdb@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 21d70161-a16f-4d54-be99-e88c83e06a79 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec e3e2e0a1-062b-480a-a483-660254e2abe0 ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 6e:ce:a9:a6:c0:4c brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec cb0e0d0a-ff37-4145-9b11-aca82942493d ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether ae:7b:c6:41:f1:08 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec d9772ef6-bc47-474a-bfcc-3a2f17bad12f ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 4e:76:20:53:9b:33 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 88ddfd64-54be-4295-97e6-4a1bac51ffeb ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 42:dd:73:98:9c:c7 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 1631d05f-a264-4956-8bd0-9851884eb00b ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:fc:2f:72:fb:29 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 21d70161-a16f-4d54-be99-e88c83e06a79 ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ba:50:43:9a:e9:50 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

#### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:60 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.11/24 brd 172.16.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:63 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 2e:ef:76:4d:74:2f brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.1/16 brd 10.85.255.255 scope global cni0
       valid_lft forever preferred_lft forever
5: veth2c3744a3@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether 76:c7:d5:bb:81:33 brd ff:ff:ff:ff:ff:ff link-netns e3e2e0a1-062b-480a-a483-660254e2abe0
6: vethf3cf048b@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether 4a:9e:48:be:23:c9 brd ff:ff:ff:ff:ff:ff link-netns cb0e0d0a-ff37-4145-9b11-aca82942493d
7: veth1d625a4b@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether 9e:09:10:dd:b5:13 brd ff:ff:ff:ff:ff:ff link-netns d9772ef6-bc47-474a-bfcc-3a2f17bad12f
8: veth0593223e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether 76:52:17:7f:a4:02 brd ff:ff:ff:ff:ff:ff link-netns 88ddfd64-54be-4295-97e6-4a1bac51ffeb
11: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 66:b9:7e:86:1e:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.0/32 scope global vxlan.calico
       valid_lft forever preferred_lft forever
12: cali9d04617f188@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 1631d05f-a264-4956-8bd0-9851884eb00b
13: cali1dac8782bdb@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns 21d70161-a16f-4d54-be99-e88c83e06a79
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec e3e2e0a1-062b-480a-a483-660254e2abe0 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 6e:ce:a9:a6:c0:4c brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 10.85.0.4/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6cce:a9ff:fea6:c04c/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec cb0e0d0a-ff37-4145-9b11-aca82942493d ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether ae:7b:c6:41:f1:08 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 10.85.0.5/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ac7b:c6ff:fe41:f108/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec d9772ef6-bc47-474a-bfcc-3a2f17bad12f ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 4e:76:20:53:9b:33 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 10.85.0.6/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::4c76:20ff:fe53:9b33/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 88ddfd64-54be-4295-97e6-4a1bac51ffeb ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 42:dd:73:98:9c:c7 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 10.85.0.7/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::40dd:73ff:fe98:9cc7/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 1631d05f-a264-4956-8bd0-9851884eb00b ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether fa:fc:2f:72:fb:29 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 192.168.2.1/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f8fc:2fff:fe72:fb29/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 21d70161-a16f-4d54-be99-e88c83e06a79 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether ba:50:43:9a:e9:50 brd ff:ff:ff:ff:ff:ff link-netns a6630dc0-bda5-4a80-8e8b-f9ed6866da34
    inet 192.168.2.2/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b850:43ff:fe9a:e950/64 scope link
       valid_lft forever preferred_lft forever
```

#### ルート

ルーティングを確認する。

```sh
ip route show
```

```
default via 172.16.0.254 dev eth0 proto static metric 100
10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.11 metric 101
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev eth0 proto kernel scope link src 172.16.0.11 metric 100
blackhole 192.168.2.0/26 proto 80
192.168.2.1 dev cali9d04617f188 scope link
192.168.2.2 dev cali1dac8782bdb scope link
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec e3e2e0a1-062b-480a-a483-660254e2abe0 ip route show
```

```
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.4
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec cb0e0d0a-ff37-4145-9b11-aca82942493d ip route show
```

```
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.5
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec d9772ef6-bc47-474a-bfcc-3a2f17bad12f ip route show
```

```
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.6
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 88ddfd64-54be-4295-97e6-4a1bac51ffeb ip route show
```

```
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.7
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 1631d05f-a264-4956-8bd0-9851884eb00b ip route show
```

```
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 21d70161-a16f-4d54-be99-e88c83e06a79 ip route show
```

```
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
```

#### iptables

フィルタを確認する。

```sh
iptables -n -L
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
cali-INPUT  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Cz_u1IQiXIMmKD4c */
KUBE-PROXY-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes load balancer firewall */
KUBE-NODEPORTS  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes health check service ports */
KUBE-EXTERNAL-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes externally-visible service portals */
KUBE-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
cali-FORWARD  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:wUHhoiAYhphO9Mso */
KUBE-PROXY-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes load balancer firewall */
KUBE-FORWARD  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes service portals */
KUBE-EXTERNAL-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes externally-visible service portals */
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:S93hcgKJrXEqnTfs */ /* Policy explicitly accepted packet. */ mark match 0x10000/0x10000
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:mp77cMpurHhyjLrM */ MARK or 0x10000

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
cali-OUTPUT  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:tVnHkvAo15HuiPy0 */
KUBE-PROXY-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes load balancer firewall */
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes service portals */
KUBE-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0

Chain KUBE-EXTERNAL-SERVICES (2 references)
target     prot opt source               destination

Chain KUBE-FIREWALL (2 references)
target     prot opt source               destination
DROP       0    -- !127.0.0.0/8          127.0.0.0/8          /* block incoming localnet connections */ ! ctstate RELATED,ESTABLISHED,DNAT

Chain KUBE-FORWARD (1 references)
target     prot opt source               destination
DROP       0    --  0.0.0.0/0            0.0.0.0/0            ctstate INVALID
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */ mark match 0x4000/0x4000
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack rule */ ctstate RELATED,ESTABLISHED

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination

Chain KUBE-NODEPORTS (1 references)
target     prot opt source               destination

Chain KUBE-PROXY-CANARY (0 references)
target     prot opt source               destination

Chain KUBE-PROXY-FIREWALL (3 references)
target     prot opt source               destination

Chain KUBE-SERVICES (2 references)
target     prot opt source               destination

Chain cali-FORWARD (1 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:vjrMJCRpqwy5oRoX */ MARK and 0xfff1ffff
cali-from-hep-forward  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:A_sPAO0mcxbT9mOV */ mark match 0x0/0x10000
cali-from-wl-dispatch  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:8ZoYfO5HKXWbB3pk */
cali-to-wl-dispatch  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:jdEuaPBe14V2hutn */
cali-to-hep-forward  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:12bc6HljsMKsmfr- */
cali-cidr-block  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:NOSxoaGx8OIstr1z */

Chain cali-INPUT (1 references)
target     prot opt source               destination
ACCEPT     17   --  0.0.0.0/0            0.0.0.0/0            /* cali:J76FwxInZIsk7uKY */ /* Allow IPv4 VXLAN packets from allowed hosts */ multiport dports 4789 match-set cali40all-vxlan-net src ADDRTYPE match dst-type LOCAL
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:EDCNTTxYfggApx8C */ /* Drop IPv4 VXLAN packets from non-allowed hosts */ multiport dports 4789 ADDRTYPE match dst-type LOCAL
cali-wl-to-host  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:H03xYXARh4e8pwd4 */
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:MN6K3isIWBigb1Va */ mark match 0x10000/0x10000
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OSYphBLwOgic22Hz */ MARK and 0xfff0ffff
cali-from-host-endpoint  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:rmi2_piRVmfeiwVp */
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:F7Q8zu44qIbOVben */ /* Host endpoint policy accepted packet. */ mark match 0x10000/0x10000

Chain cali-OUTPUT (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Mq1_rAdXXH3YkrzW */ mark match 0x10000/0x10000
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:69FkRTJDvD5Vu6Vl */
ACCEPT     17   --  0.0.0.0/0            0.0.0.0/0            /* cali:-QZG79DohFjalQBb */ /* Allow IPv4 VXLAN packets to other allowed hosts */ multiport dports 4789 ADDRTYPE match src-type LOCAL match-set cali40all-vxlan-net dst
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:iC1pSPgbvgQzkUk_ */ MARK and 0xfff0ffff
cali-to-host-endpoint  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:4Zh7KtRvt4W5AEBR */ ! ctstate DNAT
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Y0k-bqjt-5CUqyUq */ /* Host endpoint policy accepted packet. */ mark match 0x10000/0x10000

Chain cali-cidr-block (1 references)
target     prot opt source               destination

Chain cali-from-hep-forward (1 references)
target     prot opt source               destination

Chain cali-from-host-endpoint (1 references)
target     prot opt source               destination

Chain cali-from-wl-dispatch (2 references)
target     prot opt source               destination
cali-fw-cali1dac8782bdb  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:aHK7yU_mJjLvN9P9 */
cali-fw-cali3579be14420  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:wHNdnT4ytw3hnQFt */
cali-from-wl-dispatch-6  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:_b2AeThG6a53s1Jk */
cali-fw-cali868a4e4c0db  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:Bb81zxJNbVhCAzR- */
cali-fw-cali9d04617f188  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:RzmU3PGwDi5LZ3Y1 */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:CPCo32RHIaeTT15t */ /* Unknown interface */

Chain cali-from-wl-dispatch-6 (1 references)
target     prot opt source               destination
cali-fw-cali6945b3c77c1  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:51KYRHlKbzvZo9Tu */
cali-fw-cali6f60a778e0b  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:K6kzaLeHUWg36Bzo */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:BloWYvybf6fUB-zQ */ /* Unknown interface */

Chain cali-fw-cali1dac8782bdb (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ewXlV09BwxlTmEX- */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:6_-94GUCd3jeMzuW */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:gj5Txhi9uVaB6bDT */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:OPts9jGapGnT2WV8 */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:Usb6boV_eqw3tgpW */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-_kJqfZpgUe7r2t4A-14  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:03sJ_J_dL3kuHE4Q */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:TdHC3BoVwoQ7eqSG */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_4yi5_iSUAwsU8zMHTk  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:YdUyQXYlcTpzYtoj */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:LhHEycC1jQTUucMh */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:7JRFyOlI1qp70OgT */ /* Drop if no profiles matched */

Chain cali-fw-cali3579be14420 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:vvLgV90LIZd_T-gR */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:auNb1XkmPDVVzCs9 */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:UL1XRjWiDrADWTS1 */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:noK--MUYFTfT_xF- */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:EPgHCPf0LWuR4iLf */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ho1gNpaQf2FsBwTp */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:NWJs7d7MSIf483BS */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_nzzjLvInId1gPHmQz_  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:GSNV1_I3bEFXk14- */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-S1Bk-qRjicxhMqW */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:A70r47y2uGKSPVIn */ /* Drop if no profiles matched */

Chain cali-fw-cali6945b3c77c1 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-db-9r1pdOkcPeXV */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:05Cg5Hj-EgQSn3aV */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:M_24EbTTpNA31qQj */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:Dcq0ZfOzjhng02y0 */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:askR-WWM5RBQ3fIf */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-kns.kube-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:kFpZFMONqSKa7dnz */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:7gQj2O9Y2aHDO9i6 */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_u2Tn2rSoAPffvE7JO6  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Ury1iaxgqSBDRiGy */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:WAuALgJrasFzkqMB */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:2y0yOe9ix2PzV009 */ /* Drop if no profiles matched */

Chain cali-fw-cali6f60a778e0b (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:K9kICjrne1kMXosf */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OriVGB7ZF8J--YIn */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:MO1ht-ZFH-jh0esO */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:7VgLyBf2RL4Niy1W */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:dwfhFIdf_8T6CLb4 */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-kns.kube-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:agOLoOElgQCFDMVc */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:2cKBlIjOwUATyAIB */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_u2Tn2rSoAPffvE7JO6  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:9C9CUqvofk4tmgGi */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:NFpT8dCZRcHFoNRg */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:yvVNE4G3aST1Pd-g */ /* Drop if no profiles matched */

Chain cali-fw-cali868a4e4c0db (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:2GsL5Kvr52V3_Rc2 */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:5o-l0ZEH2CSK1kg1 */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:AZ82OLFQi5JrA3Ko */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:0cVjpDc4VFZSZlol */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:XvPonA1l5atzC7yv */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:hyDiszvl7n_y88Bi */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:_BuqAFFnLhFo1xa2 */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_og73BH3DuNOZrbBKFW  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:P4tcGyeiPTjzY0L_ */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:hJqGCj5uvpf4N2ti */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:BxV4qTM2roAoCXeD */ /* Drop if no profiles matched */

Chain cali-fw-cali9d04617f188 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:N40hPsqMkfQRYlJ7 */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:MxQUeXZcW5QBz8sr */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:3pjfg0PWag0UFIPV */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:XI96Wndze3ncmJa- */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:oLimpf8WdnTXhYEK */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-_kJqfZpgUe7r2t4A-14  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ak5-9EGlmo9BwZsU */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:FRZDvBhkephddg8a */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_4yi5_iSUAwsU8zMHTk  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:kvWyhkQPQ-FiFoKb */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:PBgQ9X2yoy9bFVz8 */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:P5x-QJwX54__IeQE */ /* Drop if no profiles matched */

Chain cali-pi-_3CJ_GmvE9pcCktVJ2ep (2 references)
target     prot opt source               destination
MARK       6    --  0.0.0.0/0            0.0.0.0/0            /* cali:I0yo8ky1YADcMXRf */ /* Policy calico-apiserver/knp.default.allow-apiserver ingress */ multiport dports 5443 MARK or 0x10000

Chain cali-pri-_4yi5_iSUAwsU8zMHTk (2 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ZYnaZZFwsSjfXO4C */ /* Profile ksa.calico-apiserver.calico-apiserver ingress */

Chain cali-pri-_kJqfZpgUe7r2t4A-14 (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:IQx0SzlDGn6BPv0A */ /* Profile kns.calico-apiserver ingress */ MARK or 0x10000

Chain cali-pri-_nzzjLvInId1gPHmQz_ (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:UQoEf2WCdU0bPTCb */ /* Profile ksa.calico-system.calico-kube-controllers ingress */

Chain cali-pri-_og73BH3DuNOZrbBKFW (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OzmBDJrkeSSUiXhX */ /* Profile ksa.calico-system.default ingress */

Chain cali-pri-_u2Tn2rSoAPffvE7JO6 (2 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:WqgznqAQ-uYV0oBx */ /* Profile ksa.kube-system.coredns ingress */

Chain cali-pri-kns.calico-system (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:hLANj-OVIyT53h_j */ /* Profile kns.calico-system ingress */ MARK or 0x10000

Chain cali-pri-kns.kube-system (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:J1TyxtHWd0qaBGK- */ /* Profile kns.kube-system ingress */ MARK or 0x10000

Chain cali-pro-_4yi5_iSUAwsU8zMHTk (2 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Pp_dQp2FeNabRhyi */ /* Profile ksa.calico-apiserver.calico-apiserver egress */

Chain cali-pro-_kJqfZpgUe7r2t4A-14 (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:_cFTxC141wwWRzyZ */ /* Profile kns.calico-apiserver egress */ MARK or 0x10000

Chain cali-pro-_nzzjLvInId1gPHmQz_ (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:5bHxBXLMkJKgC6dk */ /* Profile ksa.calico-system.calico-kube-controllers egress */

Chain cali-pro-_og73BH3DuNOZrbBKFW (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:09a9ZBFOA1dZ4RN5 */ /* Profile ksa.calico-system.default egress */

Chain cali-pro-_u2Tn2rSoAPffvE7JO6 (2 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:0-_UPh39dt5XfhmJ */ /* Profile ksa.kube-system.coredns egress */

Chain cali-pro-kns.calico-system (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:gWxJzCZXxl31NR0P */ /* Profile kns.calico-system egress */ MARK or 0x10000

Chain cali-pro-kns.kube-system (2 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:tgOR2S8DVHZW3F1M */ /* Profile kns.kube-system egress */ MARK or 0x10000

Chain cali-to-hep-forward (1 references)
target     prot opt source               destination

Chain cali-to-host-endpoint (1 references)
target     prot opt source               destination

Chain cali-to-wl-dispatch (1 references)
target     prot opt source               destination
cali-tw-cali1dac8782bdb  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:1PVbibbGdAYWXuD8 */
cali-tw-cali3579be14420  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:QsyNwpYieiiyOGzM */
cali-to-wl-dispatch-6  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:KujuNK2hBy3nyojz */
cali-tw-cali868a4e4c0db  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:zsKliieYuD6jYPaw */
cali-tw-cali9d04617f188  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:iNjyAaVU-t635DEz */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:M_1efBassqWauUhR */ /* Unknown interface */

Chain cali-to-wl-dispatch-6 (1 references)
target     prot opt source               destination
cali-tw-cali6945b3c77c1  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:voCtztmwuL5cVC6K */
cali-tw-cali6f60a778e0b  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:bkgw_3X7Q9UFJeX4 */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:BYUERmLynWaeKPK_ */ /* Unknown interface */

Chain cali-tw-cali1dac8782bdb (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:pZCSozePUt12JjYr */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:A2w3HQa4qF2hpRLf */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Hldf7OOaxwfrGE_R */ MARK and 0xfffcffff
cali-pi-_3CJ_GmvE9pcCktVJ2ep  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-8I24aar_tGKdtnp */ mark match 0x0/0x20000
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:j7Un6mGzIALoy9bd */ /* Return if policy accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:igRLwP2_X1SGpxjf */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
cali-pri-_kJqfZpgUe7r2t4A-14  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:7JB2IpJDPRxQwzx5 */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:W_75Fxd5jgZYCBHH */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_4yi5_iSUAwsU8zMHTk  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:6uxk--WQaBWiJbzt */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:V14_62kBGCNH6J4S */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-sAvaS7VGObTCxWi */ /* Drop if no profiles matched */

Chain cali-tw-cali3579be14420 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ZbEEMzdG5yuIEUpQ */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:xR7uPRELIucbhnex */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OsBBf9wiOTsQysXI */ MARK and 0xfffcffff
cali-pri-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:V5f8Dy0IYwFXMDZF */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:jnubSzcdhLnyl90y */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_nzzjLvInId1gPHmQz_  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-sPvnCqIWC97ZLOg */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:DP4UOlOUtmQnJzwP */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:HX7weZxPKYESRMJr */ /* Drop if no profiles matched */

Chain cali-tw-cali6945b3c77c1 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:HQmb4ynkZAbRDTQN */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Fl5WOjTZC8NjDm7g */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:bQPhfGtTyDOCRNps */ MARK and 0xfffcffff
cali-pri-kns.kube-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:o51s5VsmSAnu3N01 */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:9cr8CTQGKGqpza1O */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_u2Tn2rSoAPffvE7JO6  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:0fZCOHVuLlEfnafW */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:HjD0Bnok152LKInM */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:h3S9jIqjz-8yyKFs */ /* Drop if no profiles matched */

Chain cali-tw-cali6f60a778e0b (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:rDn_5radniFjydIo */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:MTjDb9Hu10qEBsPL */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:W7mmsTVfWMoQYec3 */ MARK and 0xfffcffff
cali-pri-kns.kube-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:PlpKu7i_1jWajWJi */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:ZEU2umto3l1NcFzv */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_u2Tn2rSoAPffvE7JO6  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:lH8KrKKnlL4YrqsH */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Ps3HdQND5pgrkIeE */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:yqVmbtSRZA6bIG5X */ /* Drop if no profiles matched */

Chain cali-tw-cali868a4e4c0db (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:LFciQPNfUKzbLPi- */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:p6C1FQj-hWgTuO41 */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:W1y5Xr5YPewj3skL */ MARK and 0xfffcffff
cali-pri-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:LtIKAIb73M9jrKUS */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:S1OrQA0emozlaTf3 */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_og73BH3DuNOZrbBKFW  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:CzKkvEj2WqqrG2Gc */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:wY8YbqtziuszMXib */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:okbPxa5a4JHLjOVp */ /* Drop if no profiles matched */

Chain cali-tw-cali9d04617f188 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:aI2Aaa5LKu14HHDJ */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:shFl6KMYyMZR-FgZ */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:S8V4Rge_tSfEyYrh */ MARK and 0xfffcffff
cali-pi-_3CJ_GmvE9pcCktVJ2ep  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:-_jakFcAn-u2FnWV */ mark match 0x0/0x20000
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:vQNIz5Om0RLh4CV6 */ /* Return if policy accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:HBvhyrmcplUpTLGt */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
cali-pri-_kJqfZpgUe7r2t4A-14  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:KRMKKLfc83tna7DN */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:guLX12dDe9wHSwcW */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_4yi5_iSUAwsU8zMHTk  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:qEJgKQc0hj9i2IqG */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:fgrNWuc2h7PDmLnk */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:r-gYK02YW5qXTOG9 */ /* Drop if no profiles matched */

Chain cali-wl-to-host (1 references)
target     prot opt source               destination
cali-from-wl-dispatch  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Ee9Sbo10IpVujdIY */
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:nSZbcOoG1xPONxb8 */ /* Configured DefaultEndpointToHostAction */
```

NAT を確認する。

```sh
iptables -t nat -n -L
```

```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
cali-PREROUTING  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:6gwbT8clXdHdC1b1 */
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
cali-OUTPUT  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:tVnHkvAo15HuiPy0 */
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
cali-POSTROUTING  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:O3lYWMrLQYEMJtB5 */
KUBE-POSTROUTING  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
CNI-0753f7d0856b570d6e276401  0    --  10.85.0.4            0.0.0.0/0            /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */
CNI-48a9f059df3e8ab377561c26  0    --  10.85.0.5            0.0.0.0/0            /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */
CNI-e2a92502cf20ebca6162a089  0    --  10.85.0.6            0.0.0.0/0            /* name: "crio" id: "518fe79a0d208707ddd4cb7a38706952641ba2c47877f5a56ced2b7532ea82db" */
CNI-6803d780133891dda2be2674  0    --  10.85.0.7            0.0.0.0/0            /* name: "crio" id: "954ae6e1a03e5e369067e14e763c869fd801b351df021cb8af46f74640554856" */

Chain CNI-0753f7d0856b570d6e276401 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */

Chain CNI-48a9f059df3e8ab377561c26 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */

Chain CNI-6803d780133891dda2be2674 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "954ae6e1a03e5e369067e14e763c869fd801b351df021cb8af46f74640554856" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "954ae6e1a03e5e369067e14e763c869fd801b351df021cb8af46f74640554856" */

Chain CNI-e2a92502cf20ebca6162a089 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "518fe79a0d208707ddd4cb7a38706952641ba2c47877f5a56ced2b7532ea82db" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "518fe79a0d208707ddd4cb7a38706952641ba2c47877f5a56ced2b7532ea82db" */

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination

Chain KUBE-MARK-MASQ (16 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            MARK or 0x4000

Chain KUBE-NODEPORTS (1 references)
target     prot opt source               destination

Chain KUBE-POSTROUTING (1 references)
target     prot opt source               destination
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000
MARK       0    --  0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
MASQUERADE  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */ random-fully

Chain KUBE-PROXY-CANARY (0 references)
target     prot opt source               destination

Chain KUBE-SEP-23Y66C2VAJ3WDEMI (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  172.16.0.11          0.0.0.0/0            /* default/kubernetes:https */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https */ tcp to:172.16.0.11:6443

Chain KUBE-SEP-63Q3QMXTSXKJR2EZ (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.4            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */ tcp to:10.85.0.4:53

Chain KUBE-SEP-A4KLBZUL4ZZDQPAH (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.5            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp */ tcp to:10.85.0.5:53

Chain KUBE-SEP-EI34DFWCIU6LMT63 (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.4            0.0.0.0/0            /* kube-system/kube-dns:metrics */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics */ tcp to:10.85.0.4:9153

Chain KUBE-SEP-HDNVQHEBMPSP33XA (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.5            0.0.0.0/0            /* kube-system/kube-dns:dns */
DNAT       17   --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns */ udp to:10.85.0.5:53

Chain KUBE-SEP-KTM2MCEYUDOLTZBE (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.5            0.0.0.0/0            /* kube-system/kube-dns:metrics */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics */ tcp to:10.85.0.5:9153

Chain KUBE-SEP-KYW2VMGBRJBFFYSZ (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  192.168.2.2          0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */ tcp to:192.168.2.2:5443

Chain KUBE-SEP-MEN2AJW7XCZSUCKL (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  192.168.2.1          0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver */ tcp to:192.168.2.1:5443

Chain KUBE-SEP-T7B34ACBXBTVP7YI (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  172.16.0.11          0.0.0.0/0            /* calico-system/calico-typha:calico-typha */
DNAT       6    --  0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha */ tcp to:172.16.0.11:5473

Chain KUBE-SEP-WAIILD3LBHUAKL5L (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.4            0.0.0.0/0            /* kube-system/kube-dns:dns */
DNAT       17   --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns */ udp to:10.85.0.4:53

Chain KUBE-SERVICES (2 references)
target     prot opt source               destination
KUBE-SVC-TCOU7JCQXEZGVUNU  17   --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
KUBE-SVC-RK657RLKDNVNU64O  6    --  0.0.0.0/0            10.110.115.144       /* calico-system/calico-typha:calico-typha cluster IP */ tcp dpt:5473
KUBE-SVC-I24EZXP75AX5E7TU  6    --  0.0.0.0/0            10.105.200.188       /* calico-apiserver/calico-api:apiserver cluster IP */ tcp dpt:443
KUBE-SVC-NPX46M4PTMTKRN6Y  6    --  0.0.0.0/0            10.96.0.1            /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SVC-ERIFXISQEP7F7OF4  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */ tcp dpt:53
KUBE-SVC-JD5MR3NA4I4DYORP  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */ tcp dpt:9153
KUBE-NODEPORTS  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL

Chain KUBE-SVC-ERIFXISQEP7F7OF4 (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */ tcp dpt:53
KUBE-SEP-63Q3QMXTSXKJR2EZ  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 10.85.0.4:53 */ statistic mode random probability 0.50000000000
KUBE-SEP-A4KLBZUL4ZZDQPAH  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 10.85.0.5:53 */

Chain KUBE-SVC-I24EZXP75AX5E7TU (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.105.200.188       /* calico-apiserver/calico-api:apiserver cluster IP */ tcp dpt:443
KUBE-SEP-MEN2AJW7XCZSUCKL  0    --  0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver -> 192.168.2.1:5443 */ statistic mode random probability 0.50000000000
KUBE-SEP-KYW2VMGBRJBFFYSZ  0    --  0.0.0.0/0            0.0.0.0/0            /* calico-apiserver/calico-api:apiserver -> 192.168.2.2:5443 */

Chain KUBE-SVC-JD5MR3NA4I4DYORP (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */ tcp dpt:9153
KUBE-SEP-EI34DFWCIU6LMT63  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 10.85.0.4:9153 */ statistic mode random probability 0.50000000000
KUBE-SEP-KTM2MCEYUDOLTZBE  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 10.85.0.5:9153 */

Chain KUBE-SVC-NPX46M4PTMTKRN6Y (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.1            /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SEP-23Y66C2VAJ3WDEMI  0    --  0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https -> 172.16.0.11:6443 */

Chain KUBE-SVC-RK657RLKDNVNU64O (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.110.115.144       /* calico-system/calico-typha:calico-typha cluster IP */ tcp dpt:5473
KUBE-SEP-T7B34ACBXBTVP7YI  0    --  0.0.0.0/0            0.0.0.0/0            /* calico-system/calico-typha:calico-typha -> 172.16.0.11:5473 */

Chain KUBE-SVC-TCOU7JCQXEZGVUNU (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  17   -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
KUBE-SEP-WAIILD3LBHUAKL5L  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 10.85.0.4:53 */ statistic mode random probability 0.50000000000
KUBE-SEP-HDNVQHEBMPSP33XA  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 10.85.0.5:53 */

Chain cali-OUTPUT (1 references)
target     prot opt source               destination
cali-fip-dnat  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:GBTAv2p5CwevEyJm */

Chain cali-POSTROUTING (1 references)
target     prot opt source               destination
cali-fip-snat  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Z-c7XtVd2Bq7s_hA */
cali-nat-outgoing  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:nYKhEzDlr11Jccal */
MASQUERADE  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:e9dnSgSVNmIcpVhP */ ADDRTYPE match src-type !LOCAL limit-out ADDRTYPE match src-type LOCAL random-fully

Chain cali-PREROUTING (1 references)
target     prot opt source               destination
cali-fip-dnat  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:r6XmIziWUJsdOK6Z */

Chain cali-fip-dnat (2 references)
target     prot opt source               destination

Chain cali-fip-snat (1 references)
target     prot opt source               destination

Chain cali-nat-outgoing (1 references)
target     prot opt source               destination
MASQUERADE  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:flqWnvo8yq4ULQLa */ match-set cali40masq-ipam-pools src ! match-set cali40all-ipam-pools dst random-fully
```

#### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all --all-namespaces
```

```
NAMESPACE          NAME                                                READY   STATUS    RESTARTS   AGE
calico-apiserver   pod/calico-apiserver-56cc46c8d5-6df76               1/1     Running   0          73m
calico-apiserver   pod/calico-apiserver-56cc46c8d5-vl57t               1/1     Running   0          73m
calico-system      pod/calico-kube-controllers-597855f5d-pvnh4         1/1     Running   0          76m
calico-system      pod/calico-node-6bn67                               1/1     Running   0          76m
calico-system      pod/calico-typha-6655bb4b48-xqtpn                   1/1     Running   0          76m
calico-system      pod/csi-node-driver-dpdzx                           2/2     Running   0          76m
kube-system        pod/coredns-7db6d8ff4d-p4kzt                        1/1     Running   1          169m
kube-system        pod/coredns-7db6d8ff4d-slnrj                        1/1     Running   1          169m
kube-system        pod/etcd-controller.home.local                      1/1     Running   1          169m
kube-system        pod/kube-apiserver-controller.home.local            1/1     Running   1          170m
kube-system        pod/kube-controller-manager-controller.home.local   1/1     Running   1          170m
kube-system        pod/kube-proxy-skh2j                                1/1     Running   1          169m
kube-system        pod/kube-scheduler-controller.home.local            1/1     Running   1          170m
tigera-operator    pod/tigera-operator-76ff79f7fd-hmsnw                1/1     Running   0          78m

NAMESPACE          NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
calico-apiserver   service/calico-api                        ClusterIP   10.105.200.188   <none>        443/TCP                  73m
calico-system      service/calico-kube-controllers-metrics   ClusterIP   None             <none>        9094/TCP                 75m
calico-system      service/calico-typha                      ClusterIP   10.110.115.144   <none>        5473/TCP                 76m
default            service/kubernetes                        ClusterIP   10.96.0.1        <none>        443/TCP                  169m
kube-system        service/kube-dns                          ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   169m

NAMESPACE       NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
calico-system   daemonset.apps/calico-node       1         1         1       1            1           kubernetes.io/os=linux   76m
calico-system   daemonset.apps/csi-node-driver   1         1         1       1            1           kubernetes.io/os=linux   76m
kube-system     daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   169m

NAMESPACE          NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-apiserver   deployment.apps/calico-apiserver          2/2     2            2           73m
calico-system      deployment.apps/calico-kube-controllers   1/1     1            1           76m
calico-system      deployment.apps/calico-typha              1/1     1            1           76m
kube-system        deployment.apps/coredns                   2/2     2            2           169m
tigera-operator    deployment.apps/tigera-operator           1/1     1            1           78m

NAMESPACE          NAME                                                DESIRED   CURRENT   READY   AGE
calico-apiserver   replicaset.apps/calico-apiserver-56cc46c8d5         2         2         2       73m
calico-system      replicaset.apps/calico-kube-controllers-597855f5d   1         1         1       76m
calico-system      replicaset.apps/calico-typha-6655bb4b48             1         1         1       76m
kube-system        replicaset.apps/coredns-7db6d8ff4d                  2         2         2       169m
tigera-operator    replicaset.apps/tigera-operator-76ff79f7fd          1         1         1       78m
```

#### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```
CONTAINER           IMAGE                                                                                                            CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
521fddde1cbef       docker.io/calico/apiserver@sha256:97ddbfa56602907922de2af0e1c2cda1052d1abadb72a9568501ca5697e4f487               About an hour ago   Running             calico-apiserver            0                   267895388d070       calico-apiserver-56cc46c8d5-vl57t
54f815203bddd       docker.io/calico/apiserver@sha256:97ddbfa56602907922de2af0e1c2cda1052d1abadb72a9568501ca5697e4f487               About an hour ago   Running             calico-apiserver            0                   9da6f82796599       calico-apiserver-56cc46c8d5-6df76
ff6bdcf4a4ea5       docker.io/calico/node@sha256:385bf6391fea031649b8575799248762a2caece86e6e3f33ffee19c0c096e6a8                    About an hour ago   Running             calico-node                 0                   0ba0fd9064057       calico-node-6bn67
a399099bdda0a       docker.io/calico/node-driver-registrar@sha256:25a00aca99eb67ca8fc6fda2888bdce0f8bb6704d71433f3daba52e72061a785   About an hour ago   Running             csi-node-driver-registrar   0                   518fe79a0d208       csi-node-driver-dpdzx
9aa8ddd6f32d8       docker.io/calico/cni@sha256:7a3a5cf6c79243ba2de9eef8cb20fac7c46ef75b858956b9884b0ce87b9a354d                     About an hour ago   Exited              install-cni                 0                   0ba0fd9064057       calico-node-6bn67
bcd70b870d86b       docker.io/calico/kube-controllers@sha256:206926f6f12f01eab89dd5f8f1e2b33b7b8f2174a85e7dab69cd9008d6062f0f        About an hour ago   Running             calico-kube-controllers     0                   954ae6e1a03e5       calico-kube-controllers-597855f5d-pvnh4
c2bfedd6b74dc       docker.io/calico/csi@sha256:6590b3466d5bfeb26e0337b85bdd25ff5361e6373e5e5a72fab5212a65c62247                     About an hour ago   Running             calico-csi                  0                   518fe79a0d208       csi-node-driver-dpdzx
135b105aa69d6       587b28ecfc62e2a60919e6a39f9b25be37c77da99d8c84252716fa3a49a171b9                                                 About an hour ago   Exited              flexvol-driver              0                   0ba0fd9064057       calico-node-6bn67
8e14c8f70853c       docker.io/calico/typha@sha256:77677c3b2614923988960151008ffc876582c722199e4b0a9a084a70b6539637                   About an hour ago   Running             calico-typha                0                   f7aaea7d48606       calico-typha-6655bb4b48-xqtpn
a6934fcf09713       quay.io/tigera/operator@sha256:479ddc7ff9ab095058b96f6710bbf070abada86332e267d6e5dcc1df36ba2cc5                  About an hour ago   Running             tigera-operator             0                   4e203c4a4ec86       tigera-operator-76ff79f7fd-hmsnw
e0b950eafce8b       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4                                                 2 hours ago         Running             coredns                     1                   b7a5dc968252f       coredns-7db6d8ff4d-p4kzt
45a457c47a13f       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4                                                 2 hours ago         Running             coredns                     1                   a0f922c5fc8fb       coredns-7db6d8ff4d-slnrj
2c8ef1733279d       747097150317f99937cabea484cff90097a2dbd79e7eb348b71dc0af879883cd                                                 2 hours ago         Running             kube-proxy                  1                   cd4e57dfd4400       kube-proxy-skh2j
1c7d83fbe5c01       a52dc94f0a91256bde86a1c3027a16336bb8fea9304f9311987066307996f035                                                 2 hours ago         Running             kube-scheduler              1                   77c0488ed357a       kube-scheduler-controller.home.local
e24f682800ae7       25a1387cdab82166df829c0b70761c10e2d2afce21a7bcf9ae4e9d71fe34ef2c                                                 2 hours ago         Running             kube-controller-manager     1                   fff5a0bd1bad8       kube-controller-manager-controller.home.local
892abdc3f86ba       91be9408031725d89ff709fdf75a7666cedbf0d8831be4581310a879a096c71a                                                 2 hours ago         Running             kube-apiserver              1                   31f69e32c888a       kube-apiserver-controller.home.local
e9164e8444789       3861cfcd7c04ccac1f062788eca39487248527ef0c0cfd477a83d7691a75a899                                                 2 hours ago         Running             etcd                        1                   fee47cf00de95       etcd-controller.home.local
```

コンテナイメージを確認する。

```sh
crictl images
```

```
IMAGE                                     TAG                 IMAGE ID            SIZE
docker.io/calico/apiserver                v3.28.0             6c07591fd1cfa       97.9MB
docker.io/calico/cni                      v3.28.0             107014d9f4c89       209MB
docker.io/calico/csi                      v3.28.0             1a094aeaf1521       18.3MB
docker.io/calico/kube-controllers         v3.28.0             428d92b022539       79.2MB
docker.io/calico/node-driver-registrar    v3.28.0             0f80feca743f4       23.5MB
docker.io/calico/node                     v3.28.0             4e42b6f329bc1       355MB
docker.io/calico/pod2daemon-flexvol       v3.28.0             587b28ecfc62e       13.4MB
docker.io/calico/typha                    v3.28.0             a9372c0f51b54       71.2MB
quay.io/tigera/operator                   v1.34.0             01249e32d0f6f       73.7MB
registry.k8s.io/coredns/coredns           v1.11.1             cbb01a7bd410d       61.2MB
registry.k8s.io/etcd                      3.5.12-0            3861cfcd7c04c       151MB
registry.k8s.io/kube-apiserver            v1.30.1             91be940803172       118MB
registry.k8s.io/kube-controller-manager   v1.30.1             25a1387cdab82       112MB
registry.k8s.io/kube-proxy                v1.30.1             747097150317f       85.9MB
registry.k8s.io/kube-scheduler            v1.30.1             a52dc94f0a912       63MB
registry.k8s.io/pause                     3.9                 e6f1816883972       750kB
```

### クライアント

Calico クライアントをインストールする。

```sh
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.0/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
```

```sh
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.0/calicoctl-linux-amd64 -o kubectl-calico
chmod +x kubectl-calico
```
