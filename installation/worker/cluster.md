# クラスタ

## クラスタに参加

クラスタにワーカとして参加する。

```sh
kubeadm join controller.home.local:6443 \
    --token uko9ek.0g7wje5trztdsc8d \
    --discovery-token-ca-cert-hash sha256:a9e19ff35745b233a452c780fa4f73fbd6b1e7927ad4dfcd95d17ce40ba7b755
```

```text
[preflight] Running pre-flight checks
        [WARNING FileExisting-socat]: socat not found in system path
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 10.501804651s
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

マシン起動時の自動起動を設定する。

```sh
systemctl enable kubelet
```

## 環境確認

Worker Node でネットワーク構成を確認する。

![Worker ノードの構成](../../_static/image/controller_worker.png "Worker ノードの構成")

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```text
482f2d0f-575e-4e27-8710-a47af2823dd1
20be9af8-e917-4ab8-a5e9-b515c3804118 (id: 0)
2abf8663-2a57-4965-9f1b-86b7e37f60e9
```

### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:cf:af:cd brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio0
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:9d:d6:e9 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio1
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether a6:71:b3:c9:b1:b2 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.a6:71:b3:c9:b1:b2 designated_root 8000.a6:71:b3:c9:b1:b2 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer    4.92 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
5: vethaf1ac987@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether c2:a2:1b:1e:b3:a9 brd ff:ff:ff:ff:ff:ff link-netns 20be9af8-e917-4ab8-a5e9-b515c3804118 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.a6:71:b3:c9:b1:b2 designated_root 8000.a6:71:b3:c9:b1:b2 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:ba:9d:ce:21:bb brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    vxlan id 4096 local 10.0.0.31 dev enp2s0 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 20be9af8-e917-4ab8-a5e9-b515c3804118 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:c8:c1:30:f0:1d brd ff:ff:ff:ff:ff:ff link-netns 482f2d0f-575e-4e27-8710-a47af2823dd1 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```text
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
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 20be9af8-e917-4ab8-a5e9-b515c3804118 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:c8:c1:30:f0:1d brd ff:ff:ff:ff:ff:ff link-netns 482f2d0f-575e-4e27-8710-a47af2823dd1
    inet 10.85.0.2/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::c8:c1ff:fe30:f01d/64 scope link
       valid_lft forever preferred_lft forever
```

### ルート

ルーティングを確認する。

```sh
ip route show
```

```text
default via 172.16.0.254 dev enp1s0 proto static metric 100
10.0.0.0/24 dev enp2s0 proto kernel scope link src 10.0.0.31 metric 101
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev enp1s0 proto kernel scope link src 172.16.0.31 metric 100
172.17.2.0/26 via 10.0.0.11 dev enp2s0 proto 80 onlink
blackhole 172.17.255.128/26 proto 80
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 20be9af8-e917-4ab8-a5e9-b515c3804118 ip route show
```

```text
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.2
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

        chain cali-to-host-endpoint {
        }

        chain cali-PREROUTING {
                 ct state related,established counter packets 3242 bytes 784836 accept
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 50 bytes 3000 jump cali-from-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-from-host-endpoint {
        }

        chain cali-POSTROUTING {
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
                 counter packets 2956 bytes 433508 meta mark set mark and 0xfff0ffff
                 ct status dnat counter packets 54 bytes 3276 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 return
        }

        chain PREROUTING {
                type filter hook prerouting priority mangle; policy accept;
                 counter packets 3292 bytes 787836 jump cali-PREROUTING
        }

        chain POSTROUTING {
                type filter hook postrouting priority mangle; policy accept;
                 counter packets 2956 bytes 433508 jump cali-POSTROUTING
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain KUBE-FIREWALL {
                ip saddr != 127.0.0.0/8 ip daddr 127.0.0.0/8  ct status dnat counter packets 0 bytes 0 drop
        }

        chain OUTPUT {
                type filter hook output priority filter; policy accept;
                 counter packets 2956 bytes 433508 jump cali-OUTPUT
                ct state new  counter packets 925 bytes 57614 jump KUBE-PROXY-FIREWALL
                ct state new  counter packets 925 bytes 57614 jump KUBE-SERVICES
                counter packets 75160 bytes 4584995 jump KUBE-FIREWALL
        }

        chain INPUT {
                type filter hook input priority filter; policy accept;
                 counter packets 3292 bytes 787836 jump cali-INPUT
                ct state new  counter packets 50 bytes 3000 jump KUBE-PROXY-FIREWALL
                 counter packets 165265 bytes 243091024 jump KUBE-NODEPORTS
                ct state new  counter packets 50 bytes 3000 jump KUBE-EXTERNAL-SERVICES
                counter packets 187424 bytes 275650808 jump KUBE-FIREWALL
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-EXTERNAL-SERVICES {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                 counter packets 0 bytes 0 jump cali-FORWARD
                ct state new  counter packets 0 bytes 0 jump KUBE-PROXY-FIREWALL
                 counter packets 0 bytes 0 jump KUBE-FORWARD
                ct state new  counter packets 0 bytes 0 jump KUBE-SERVICES
                ct state new  counter packets 0 bytes 0 jump KUBE-EXTERNAL-SERVICES
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
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

        chain cali-INPUT {
                meta l4proto udp   udp dport 4789 xt match "set" fib daddr type local counter packets 0 bytes 0 accept
                meta l4proto udp   udp dport 4789 fib daddr type local counter packets 0 bytes 0 drop
                iifname "cali*"  counter packets 0 bytes 0 goto cali-wl-to-host
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                 counter packets 3292 bytes 787836 meta mark set mark and 0xfff0ffff
                 counter packets 3292 bytes 787836 jump cali-from-host-endpoint
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
                 counter packets 0 bytes 0 meta mark set mark and 0xfff1ffff
                 meta mark & 0x00010000 == 0x00000000 counter packets 0 bytes 0 jump cali-from-hep-forward
                iifname "cali*"  counter packets 0 bytes 0 jump cali-from-wl-dispatch
                oifname "cali*"  counter packets 0 bytes 0 jump cali-to-wl-dispatch
                 counter packets 0 bytes 0 jump cali-to-hep-forward
                 counter packets 0 bytes 0 jump cali-cidr-block
        }

        chain cali-wl-to-host {
                 counter packets 0 bytes 0 jump cali-from-wl-dispatch
                  counter packets 0 bytes 0 accept
        }

        chain cali-OUTPUT {
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
                oifname "cali*"  counter packets 0 bytes 0 return
                meta l4proto udp   udp dport 4789 fib saddr type local xt match "set" counter packets 0 bytes 0 accept
                 counter packets 2956 bytes 433508 meta mark set mark and 0xfff0ffff
                 ct status dnat counter packets 2902 bytes 430232 jump cali-to-host-endpoint
                  meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-pro-_og73BH3DuNOZrbBKFW {
                  counter packets 0 bytes 0
        }

        chain cali-from-wl-dispatch {
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
                 counter packets 720 bytes 43849 jump cali-POSTROUTING
                 counter packets 811 bytes 50209 jump KUBE-POSTROUTING
                ip saddr 10.85.0.2  counter packets 0 bytes 0 jump CNI-dc86429b457d5c3c87400258
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-SERVICES {
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-ERIFXISQEP7F7OF4
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-SVC-JD5MR3NA4I4DYORP
                meta l4proto tcp ip daddr 10.107.6.141  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-I24EZXP75AX5E7TU
                meta l4proto tcp ip daddr 10.105.183.5  tcp dport 5473 counter packets 0 bytes 0 jump KUBE-SVC-RK657RLKDNVNU64O
                meta l4proto tcp ip daddr 10.96.0.1  tcp dport 443 counter packets 9 bytes 540 jump KUBE-SVC-NPX46M4PTMTKRN6Y
                meta l4proto udp ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-TCOU7JCQXEZGVUNU
                 fib daddr type local counter packets 50 bytes 3000 jump KUBE-NODEPORTS
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                 counter packets 720 bytes 43849 jump cali-OUTPUT
                 counter packets 811 bytes 50209 jump KUBE-SERVICES
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                 counter packets 0 bytes 0 jump cali-PREROUTING
                 counter packets 0 bytes 0 jump KUBE-SERVICES
        }

        chain KUBE-POSTROUTING {
                meta mark & 0x00004000 != 0x00004000 counter packets 802 bytes 49669 return
                counter packets 9 bytes 540 meta mark set mark xor 0x4000
                 counter packets 9 bytes 540 masquerade fully-random
        }

        chain KUBE-NODEPORTS {
        }

        chain KUBE-MARK-MASQ {
                counter packets 9 bytes 540 meta mark set mark or 0x4000
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
                 counter packets 0 bytes 0 jump KUBE-SEP-T7B34ACBXBTVP7YI
        }

        chain KUBE-SEP-T7B34ACBXBTVP7YI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 0 bytes 0 dnat to 172.16.0.11:5473
        }

        chain KUBE-SVC-NPX46M4PTMTKRN6Y {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.1  tcp dport 443 counter packets 9 bytes 540 jump KUBE-MARK-MASQ
                 counter packets 9 bytes 540 jump KUBE-SEP-23Y66C2VAJ3WDEMI
        }

        chain KUBE-SEP-23Y66C2VAJ3WDEMI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp   counter packets 9 bytes 540 dnat to 172.16.0.11:6443
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
                 counter packets 0 bytes 0 jump cali-fip-dnat
        }

        chain cali-fip-dnat {
        }

        chain cali-POSTROUTING {
                 counter packets 720 bytes 43849 jump cali-fip-snat
                 counter packets 720 bytes 43849 jump cali-nat-outgoing
                oifname "vxlan.calico"  fib saddr . oif type != local fib saddr type local counter packets 0 bytes 0 masquerade fully-random
        }

        chain cali-fip-snat {
        }

        chain cali-nat-outgoing {
                 xt match "set" xt match "set" counter packets 0 bytes 0 masquerade fully-random
        }

        chain cali-OUTPUT {
                 counter packets 720 bytes 43849 jump cali-fip-dnat
        }
}
# Warning: table ip raw is managed by iptables-nft, do not touch!
table ip raw {
        chain cali-PREROUTING {
                 counter packets 3292 bytes 787836 meta mark set mark and 0xfff0ffff
                iifname "cali*"  counter packets 0 bytes 0 meta mark set mark or 0x40000
                 meta mark & 0x00040000 == 0x00040000 counter packets 0 bytes 0 jump cali-rpf-skip
                 meta mark & 0x00040000 == 0x00040000 fib saddr . mark . iif oif 0 counter packets 0 bytes 0 drop
                 meta mark & 0x00040000 == 0x00000000 counter packets 3292 bytes 787836 jump cali-from-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-rpf-skip {
        }

        chain cali-from-host-endpoint {
        }

        chain cali-OUTPUT {
                 counter packets 2956 bytes 433508 meta mark set mark and 0xfff0ffff
                 counter packets 2956 bytes 433508 jump cali-to-host-endpoint
                 meta mark & 0x00010000 == 0x00010000 counter packets 0 bytes 0 accept
        }

        chain cali-to-host-endpoint {
        }

        chain PREROUTING {
                type filter hook prerouting priority raw; policy accept;
                 counter packets 3292 bytes 787836 jump cali-PREROUTING
        }

        chain OUTPUT {
                type filter hook output priority raw; policy accept;
                 counter packets 2956 bytes 433508 jump cali-OUTPUT
        }
}
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all --all-namespaces -o wide
```

```text
NAMESPACE          NAME                                                READY   STATUS    RESTARTS   AGE   IP            NODE                    NOMINATED NODE   READINESS GATES
calico-apiserver   pod/calico-apiserver-556c4b7b69-h9mng               1/1     Running   1          17h   172.17.2.12   controller.home.local   <none>           <none>
calico-apiserver   pod/calico-apiserver-556c4b7b69-tdbq5               1/1     Running   1          17h   172.17.2.7    controller.home.local   <none>           <none>
calico-system      pod/calico-kube-controllers-9b679fd48-8kzvc         1/1     Running   1          17h   172.17.2.9    controller.home.local   <none>           <none>
calico-system      pod/calico-node-n84qn                               0/1     Running   0          21m   172.16.0.31   worker01.home.local     <none>           <none>
calico-system      pod/calico-node-xhj6c                               0/1     Running   1          17h   172.16.0.11   controller.home.local   <none>           <none>
calico-system      pod/calico-typha-865684d4d7-rdgqp                   1/1     Running   1          17h   172.16.0.11   controller.home.local   <none>           <none>
calico-system      pod/csi-node-driver-rzkl5                           2/2     Running   0          21m   10.85.0.2     worker01.home.local     <none>           <none>
calico-system      pod/csi-node-driver-zsc2j                           2/2     Running   2          17h   172.17.2.10   controller.home.local   <none>           <none>
kube-system        pod/coredns-7c65d6cfc9-sr5mk                        1/1     Running   1          19h   172.17.2.11   controller.home.local   <none>           <none>
kube-system        pod/coredns-7c65d6cfc9-zczmd                        1/1     Running   1          19h   172.17.2.8    controller.home.local   <none>           <none>
kube-system        pod/etcd-controller.home.local                      1/1     Running   1          19h   172.16.0.11   controller.home.local   <none>           <none>
kube-system        pod/kube-apiserver-controller.home.local            1/1     Running   1          19h   172.16.0.11   controller.home.local   <none>           <none>
kube-system        pod/kube-controller-manager-controller.home.local   1/1     Running   1          19h   172.16.0.11   controller.home.local   <none>           <none>
kube-system        pod/kube-proxy-77ff5                                1/1     Running   0          21m   172.16.0.31   worker01.home.local     <none>           <none>
kube-system        pod/kube-proxy-7zqsq                                1/1     Running   1          19h   172.16.0.11   controller.home.local   <none>           <none>
kube-system        pod/kube-scheduler-controller.home.local            1/1     Running   1          19h   172.16.0.11   controller.home.local   <none>           <none>
tigera-operator    pod/tigera-operator-6847585ccf-rwszb                1/1     Running   1          17h   172.16.0.11   controller.home.local   <none>           <none>

NAMESPACE          NAME                                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
calico-apiserver   service/calico-api                        ClusterIP   10.107.6.141   <none>        443/TCP                  17h   apiserver=true
calico-system      service/calico-kube-controllers-metrics   ClusterIP   None           <none>        9094/TCP                 17h   k8s-app=calico-kube-controllers
calico-system      service/calico-typha                      ClusterIP   10.105.183.5   <none>        5473/TCP                 17h   k8s-app=calico-typha
default            service/kubernetes                        ClusterIP   10.96.0.1      <none>        443/TCP                  19h   <none>
kube-system        service/kube-dns                          ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   19h   k8s-app=kube-dns

NAMESPACE       NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS                             IMAGES                                                                        SELECTOR
calico-system   daemonset.apps/calico-node       2         2         0       2            0           kubernetes.io/os=linux   17h   calico-node                            docker.io/calico/node:v3.28.0                                                 k8s-app=calico-node
calico-system   daemonset.apps/csi-node-driver   2         2         2       2            2           kubernetes.io/os=linux   17h   calico-csi,csi-node-driver-registrar   docker.io/calico/csi:v3.28.0,docker.io/calico/node-driver-registrar:v3.28.0   k8s-app=csi-node-driver
kube-system     daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   19h   kube-proxy                             registry.k8s.io/kube-proxy:v1.31.0                                            k8s-app=kube-proxy

NAMESPACE          NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                IMAGES                                      SELECTOR
calico-apiserver   deployment.apps/calico-apiserver          2/2     2            2           17h   calico-apiserver          docker.io/calico/apiserver:v3.28.0          apiserver=true
calico-system      deployment.apps/calico-kube-controllers   1/1     1            1           17h   calico-kube-controllers   docker.io/calico/kube-controllers:v3.28.0   k8s-app=calico-kube-controllers
calico-system      deployment.apps/calico-typha              1/1     1            1           17h   calico-typha              docker.io/calico/typha:v3.28.0              k8s-app=calico-typha
kube-system        deployment.apps/coredns                   2/2     2            2           19h   coredns                   registry.k8s.io/coredns/coredns:v1.11.3     k8s-app=kube-dns
tigera-operator    deployment.apps/tigera-operator           1/1     1            1           17h   tigera-operator           quay.io/tigera/operator:v1.34.0             name=tigera-operator

NAMESPACE          NAME                                                DESIRED   CURRENT   READY   AGE   CONTAINERS                IMAGES                                      SELECTOR
calico-apiserver   replicaset.apps/calico-apiserver-556c4b7b69         2         2         2       17h   calico-apiserver          docker.io/calico/apiserver:v3.28.0          apiserver=true,pod-template-hash=556c4b7b69
calico-system      replicaset.apps/calico-kube-controllers-9b679fd48   1         1         1       17h   calico-kube-controllers   docker.io/calico/kube-controllers:v3.28.0   k8s-app=calico-kube-controllers,pod-template-hash=9b679fd48
calico-system      replicaset.apps/calico-typha-865684d4d7             1         1         1       17h   calico-typha              docker.io/calico/typha:v3.28.0              k8s-app=calico-typha,pod-template-hash=865684d4d7
kube-system        replicaset.apps/coredns-7c65d6cfc9                  2         2         2       19h   coredns                   registry.k8s.io/coredns/coredns:v1.11.3     k8s-app=kube-dns,pod-template-hash=7c65d6cfc9
tigera-operator    replicaset.apps/tigera-operator-6847585ccf          1         1         1       17h   tigera-operator           quay.io/tigera/operator:v1.34.0             name=tigera-operator,pod-template-hash=6847585ccf
```

ノードを確認する。

```sh
kubectl get nodes
```

```text
NAME                    STATUS   ROLES           AGE   VERSION
controller.home.local   Ready    control-plane   19h   v1.31.1
worker01.home.local     Ready    <none>          22m   v1.31.1
```

### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```text
CONTAINER           IMAGE                                                                                                            CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
1e09f4362c597       docker.io/calico/node@sha256:3c0e24adecc39e89780e807400def972deb0ec9de9fbbbaceade072a8c6ae94f                    22 minutes ago      Running             calico-node                 0                   2bbadeecd8d63       calico-node-n84qn
f8ee675b2a337       docker.io/calico/cni@sha256:7a3a5cf6c79243ba2de9eef8cb20fac7c46ef75b858956b9884b0ce87b9a354d                     22 minutes ago      Exited              install-cni                 0                   2bbadeecd8d63       calico-node-n84qn
5e00e077f45c2       docker.io/calico/node-driver-registrar@sha256:f44b460f2684d33042970a7703c2b3bfa0c026980528e3b582833baf15384486   23 minutes ago      Running             csi-node-driver-registrar   0                   5dac55736598c       csi-node-driver-rzkl5
d72e234db13b1       docker.io/calico/pod2daemon-flexvol@sha256:cbb5788040c97b63dbdb9cadea3b4937ff9069d0cdbb7e9c04671583da7452a6      23 minutes ago      Exited              flexvol-driver              0                   2bbadeecd8d63       calico-node-n84qn
92c3b3a4a6aea       docker.io/calico/csi@sha256:8375b443b2b51f04eed6d29e889bc4ce7e30d9d88a26608579de22d691473a3f                     23 minutes ago      Running             calico-csi                  0                   5dac55736598c       csi-node-driver-rzkl5
cffa497347532       registry.k8s.io/kube-proxy@sha256:c31eb37ccda83c6c8ee2ee5030a7038b04ecaa393d14cb71f01ab18147366fbf               23 minutes ago      Running             kube-proxy                  0                   12711152b28fd       kube-proxy-77ff5
```

calico-node が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 1e09f4362c597 | jq '.info.pid')
```

```text
2abf8663-2a57-4965-9f1b-86b7e37f60e9
```

csi-node-driver-registrar が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 5e00e077f45c2 | jq '.info.pid')
```

```text
20be9af8-e917-4ab8-a5e9-b515c3804118
```

calico-csi が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 92c3b3a4a6aea | jq '.info.pid')
```

```text
20be9af8-e917-4ab8-a5e9-b515c3804118
```

kube-proxy が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect cffa497347532 | jq '.info.pid')
```

```text
2abf8663-2a57-4965-9f1b-86b7e37f60e9
```

コンテナイメージを確認する。

```sh
crictl images
```

```text
IMAGE                                    TAG                 IMAGE ID            SIZE
docker.io/calico/cni                     v3.28.0             107014d9f4c89       209MB
docker.io/calico/csi                     v3.28.0             1a094aeaf1521       18.3MB
docker.io/calico/node-driver-registrar   v3.28.0             0f80feca743f4       23.5MB
docker.io/calico/node                    v3.28.0             4e42b6f329bc1       355MB
docker.io/calico/pod2daemon-flexvol      v3.28.0             587b28ecfc62e       13.4MB
registry.k8s.io/kube-proxy               v1.31.0             ad83b2ca7b09e       92.7MB
registry.k8s.io/pause                    3.10                873ed75102791       742kB
```

### BGP

ペアリングを確認する。

```sh
kubectl calico node status
```

```text
```

### VXLAN

フォワーディングテーブルを確認する。

```sh
bridge fdb show dev vxlan.calico
```

```text
```
