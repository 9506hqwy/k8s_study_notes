# クラスタ

## クラスタに参加

クラスタにワーカとして参加する。

```sh
kubeadm join controller.home.local:6443 \
    --token 50xk2g.pgg67ykupqov7nrx \
    --discovery-token-ca-cert-hash sha256:8afa26c573f015cbd81c9e320d33111fbd9d6e0a43a1cf6024976de1529c6d8b
```

```
[preflight] Running pre-flight checks
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.003866074s
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

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```
dffe6da4-36e9-4690-b16b-2b29976b771b (id: 0)
de960bd8-3085-4d69-b0fa-30a7596f3ef2
420b336c-6238-44c7-b2fa-c432c2e18c0f
```

### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:61 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev afc87a0e-14ac-4286-84a4-ad8df544f564
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:64 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 14637967-e998-49d8-a914-8d80af2901e7
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 36:5c:a0:7c:00:f3 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.36:5c:a0:7c:0:f3 designated_root 8000.36:5c:a0:7c:0:f3 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  153.56 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
5: veth94fc2466@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 82:e7:0e:0b:04:6a brd ff:ff:ff:ff:ff:ff link-netns dffe6da4-36e9-4690-b16b-2b29976b771b promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.36:5c:a0:7c:0:f3 designated_root 8000.36:5c:a0:7c:0:f3 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 66:ba:9d:ce:21:bb brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    vxlan id 4096 local 10.0.0.31 dev eth1 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec dffe6da4-36e9-4690-b16b-2b29976b771b ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 3e:0e:99:04:2c:1e brd ff:ff:ff:ff:ff:ff link-netns de960bd8-3085-4d69-b0fa-30a7596f3ef2 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
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
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:61 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.31/24 brd 172.16.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:64 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.31/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 36:5c:a0:7c:00:f3 brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.1/16 brd 10.85.255.255 scope global cni0
       valid_lft forever preferred_lft forever
5: veth94fc2466@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default
    link/ether 82:e7:0e:0b:04:6a brd ff:ff:ff:ff:ff:ff link-netns dffe6da4-36e9-4690-b16b-2b29976b771b
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 66:ba:9d:ce:21:bb brd ff:ff:ff:ff:ff:ff
    inet 192.168.255.128/32 scope global vxlan.calico
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec dffe6da4-36e9-4690-b16b-2b29976b771b ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 3e:0e:99:04:2c:1e brd ff:ff:ff:ff:ff:ff link-netns de960bd8-3085-4d69-b0fa-30a7596f3ef2
    inet 10.85.0.2/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::3c0e:99ff:fe04:2c1e/64 scope link
       valid_lft forever preferred_lft forever
```

### ルート

ルーティングを確認する。

```sh
ip route show
```

```
default via 172.16.0.254 dev eth0 proto static metric 100
10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.31 metric 101
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev eth0 proto kernel scope link src 172.16.0.31 metric 100
192.168.2.0/26 via 10.0.0.11 dev eth1 proto 80 onlink
blackhole 192.168.255.128/26 proto 80
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec dffe6da4-36e9-4690-b16b-2b29976b771b ip route show
```

```
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.2
```

### iptables

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
cali-fw-cali839f4f93e61  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:SrG1j21I2siK_yQD */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:S20STpYL4dv_pcZY */ /* Unknown interface */

Chain cali-fw-cali839f4f93e61 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OMdV1VmYvgay0Ju_ */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:9GTJTvgFoQSuHKyd */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:z8vZgauvdvTX14jS */ MARK and 0xfffcffff
DROP       17   --  0.0.0.0/0            0.0.0.0/0            /* cali:WJxdljn16dqhpgmF */ /* Drop VXLAN encapped packets originating in workloads */ multiport dports 4789
DROP       4    --  0.0.0.0/0            0.0.0.0/0            /* cali:rzce_8ES3RZZ9abT */ /* Drop IPinIP encapped packets originating in workloads */
cali-pro-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:cr3QRQN7OZshnwZM */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:yQFK3WKtWLN-RIEH */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pro-_og73BH3DuNOZrbBKFW  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:bGNGJdfsMA4VCnAZ */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:4jKZ60Lo3ApMmZll */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:tzuXKpDyOxEZ7V_o */ /* Drop if no profiles matched */

Chain cali-pri-_og73BH3DuNOZrbBKFW (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:OzmBDJrkeSSUiXhX */ /* Profile ksa.calico-system.default ingress */

Chain cali-pri-kns.calico-system (1 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:hLANj-OVIyT53h_j */ /* Profile kns.calico-system ingress */ MARK or 0x10000

Chain cali-pro-_og73BH3DuNOZrbBKFW (1 references)
target     prot opt source               destination
           0    --  0.0.0.0/0            0.0.0.0/0            /* cali:09a9ZBFOA1dZ4RN5 */ /* Profile ksa.calico-system.default egress */

Chain cali-pro-kns.calico-system (1 references)
target     prot opt source               destination
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:gWxJzCZXxl31NR0P */ /* Profile kns.calico-system egress */ MARK or 0x10000

Chain cali-to-hep-forward (1 references)
target     prot opt source               destination

Chain cali-to-host-endpoint (1 references)
target     prot opt source               destination

Chain cali-to-wl-dispatch (1 references)
target     prot opt source               destination
cali-tw-cali839f4f93e61  0    --  0.0.0.0/0            0.0.0.0/0           [goto]  /* cali:4Fqv_-9RZ2vgyLDb */
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:1Dz_2BWm_qJtjSng */ /* Unknown interface */

Chain cali-tw-cali839f4f93e61 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:aAGPL5ce-sfWvtEi */ ctstate RELATED,ESTABLISHED
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:uGYjECSJP78AgcRn */ ctstate INVALID
MARK       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:MwKebX5wAu_THC67 */ MARK and 0xfffcffff
cali-pri-kns.calico-system  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:AgNwg8ifzTwYQcWu */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:7Wl-PUKK1Kcq6eU- */ /* Return if profile accepted */ mark match 0x10000/0x10000
cali-pri-_og73BH3DuNOZrbBKFW  0    --  0.0.0.0/0            0.0.0.0/0            /* cali:Kn5_KGJdoAzoXq9Q */
RETURN     0    --  0.0.0.0/0            0.0.0.0/0            /* cali:xHcktBvIC6Kjdl7A */ /* Return if profile accepted */ mark match 0x10000/0x10000
DROP       0    --  0.0.0.0/0            0.0.0.0/0            /* cali:hHYkwi_qPKC7uR09 */ /* Drop if no profiles matched */

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
CNI-cbc6716873dccdc09c13edd3  0    --  10.85.0.2            0.0.0.0/0            /* name: "crio" id: "8d0a1da018292e59b69995620c9da2793be85054fc4fe1b6cb2a98a68ad4a4ca" */

Chain CNI-cbc6716873dccdc09c13edd3 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "8d0a1da018292e59b69995620c9da2793be85054fc4fe1b6cb2a98a68ad4a4ca" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "8d0a1da018292e59b69995620c9da2793be85054fc4fe1b6cb2a98a68ad4a4ca" */

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
KUBE-SVC-RK657RLKDNVNU64O  6    --  0.0.0.0/0            10.110.115.144       /* calico-system/calico-typha:calico-typha cluster IP */ tcp dpt:5473
KUBE-SVC-NPX46M4PTMTKRN6Y  6    --  0.0.0.0/0            10.96.0.1            /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SVC-TCOU7JCQXEZGVUNU  17   --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
KUBE-SVC-ERIFXISQEP7F7OF4  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */ tcp dpt:53
KUBE-SVC-JD5MR3NA4I4DYORP  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */ tcp dpt:9153
KUBE-SVC-I24EZXP75AX5E7TU  6    --  0.0.0.0/0            10.105.200.188       /* calico-apiserver/calico-api:apiserver cluster IP */ tcp dpt:443
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

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all --all-namespaces
```

```
NAMESPACE          NAME                                                READY   STATUS    RESTARTS        AGE
calico-apiserver   pod/calico-apiserver-56cc46c8d5-6df76               1/1     Running   0               104m
calico-apiserver   pod/calico-apiserver-56cc46c8d5-vl57t               1/1     Running   0               104m
calico-system      pod/calico-kube-controllers-597855f5d-pvnh4         1/1     Running   0               107m
calico-system      pod/calico-node-6bn67                               0/1     Running   0               107m
calico-system      pod/calico-node-dvv2b                               0/1     Running   1 (5m13s ago)   7m2s
calico-system      pod/calico-typha-6655bb4b48-xqtpn                   1/1     Running   0               107m
calico-system      pod/csi-node-driver-dpdzx                           2/2     Running   0               107m
calico-system      pod/csi-node-driver-hslhz                           2/2     Running   0               7m2s
kube-system        pod/coredns-7db6d8ff4d-p4kzt                        1/1     Running   1               3h20m
kube-system        pod/coredns-7db6d8ff4d-slnrj                        1/1     Running   1               3h20m
kube-system        pod/etcd-controller.home.local                      1/1     Running   1               3h20m
kube-system        pod/kube-apiserver-controller.home.local            1/1     Running   1               3h21m
kube-system        pod/kube-controller-manager-controller.home.local   1/1     Running   1               3h21m
kube-system        pod/kube-proxy-kfqdc                                1/1     Running   0               7m2s
kube-system        pod/kube-proxy-skh2j                                1/1     Running   1               3h20m
kube-system        pod/kube-scheduler-controller.home.local            1/1     Running   1               3h21m
tigera-operator    pod/tigera-operator-76ff79f7fd-hmsnw                1/1     Running   0               109m

NAMESPACE          NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
calico-apiserver   service/calico-api                        ClusterIP   10.105.200.188   <none>        443/TCP                  104m
calico-system      service/calico-kube-controllers-metrics   ClusterIP   None             <none>        9094/TCP                 106m
calico-system      service/calico-typha                      ClusterIP   10.110.115.144   <none>        5473/TCP                 107m
default            service/kubernetes                        ClusterIP   10.96.0.1        <none>        443/TCP                  3h20m
kube-system        service/kube-dns                          ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   3h20m

NAMESPACE       NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
calico-system   daemonset.apps/calico-node       2         2         0       2            0           kubernetes.io/os=linux   107m
calico-system   daemonset.apps/csi-node-driver   2         2         2       2            2           kubernetes.io/os=linux   107m
kube-system     daemonset.apps/kube-proxy        2         2         2       2            2           kubernetes.io/os=linux   3h20m

NAMESPACE          NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-apiserver   deployment.apps/calico-apiserver          2/2     2            2           104m
calico-system      deployment.apps/calico-kube-controllers   1/1     1            1           107m
calico-system      deployment.apps/calico-typha              1/1     1            1           107m
kube-system        deployment.apps/coredns                   2/2     2            2           3h20m
tigera-operator    deployment.apps/tigera-operator           1/1     1            1           109m

NAMESPACE          NAME                                                DESIRED   CURRENT   READY   AGE
calico-apiserver   replicaset.apps/calico-apiserver-56cc46c8d5         2         2         2       104m
calico-system      replicaset.apps/calico-kube-controllers-597855f5d   1         1         1       107m
calico-system      replicaset.apps/calico-typha-6655bb4b48             1         1         1       107m
kube-system        replicaset.apps/coredns-7db6d8ff4d                  2         2         2       3h20m
tigera-operator    replicaset.apps/tigera-operator-76ff79f7fd          1         1         1       109m
```

ノードを確認する。

```sh
kubectl get nodes
```

```
NAME                    STATUS   ROLES           AGE     VERSION
controller.home.local   Ready    control-plane   3h21m   v1.30.1
worker01.home.local     Ready    <none>          7m16s   v1.30.1
```

### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```
CONTAINER           IMAGE                                                                                                            CREATED             STATE               NAME                        ATTEMPT             POD ID              POD
f98e4151fdb9e       4e42b6f329bc1d197d97f6d2a1289b9e9f4a9560db3a36c8cffb5e95e64e4b49                                                 5 minutes ago       Running             calico-node                 1                   cc59922ee227e       calico-node-dvv2b
1ab7bd602c2a3       docker.io/calico/node@sha256:385bf6391fea031649b8575799248762a2caece86e6e3f33ffee19c0c096e6a8                    5 minutes ago       Exited              calico-node                 0                   cc59922ee227e       calico-node-dvv2b
813a9c3bd90cd       docker.io/calico/node-driver-registrar@sha256:25a00aca99eb67ca8fc6fda2888bdce0f8bb6704d71433f3daba52e72061a785   6 minutes ago       Running             csi-node-driver-registrar   0                   8d0a1da018292       csi-node-driver-hslhz
0676a2a0f91c6       docker.io/calico/cni@sha256:7a3a5cf6c79243ba2de9eef8cb20fac7c46ef75b858956b9884b0ce87b9a354d                     6 minutes ago       Exited              install-cni                 0                   cc59922ee227e       calico-node-dvv2b
a0df7679b42b2       docker.io/calico/csi@sha256:6590b3466d5bfeb26e0337b85bdd25ff5361e6373e5e5a72fab5212a65c62247                     6 minutes ago       Running             calico-csi                  0                   8d0a1da018292       csi-node-driver-hslhz
a98b358fb8e07       docker.io/calico/pod2daemon-flexvol@sha256:2054fc9485e11bdde7ec8e22bca88bbf3a0f777f6c17509045a427294aa0a54b      7 minutes ago       Exited              flexvol-driver              0                   cc59922ee227e       calico-node-dvv2b
2df8e7a47d2aa       registry.k8s.io/kube-proxy@sha256:2eec8116ed9b8f46b6a90a46434711354d2222575ab50a4aca42bb6ab19989fa               7 minutes ago       Running             kube-proxy                  0                   b03b5a05e5dab       kube-proxy-kfqdc
```

コンテナイメージを確認する。

```sh
crictl images
```

```
IMAGE                                    TAG                 IMAGE ID            SIZE
docker.io/calico/cni                     v3.28.0             107014d9f4c89       209MB
docker.io/calico/csi                     v3.28.0             1a094aeaf1521       18.3MB
docker.io/calico/node-driver-registrar   v3.28.0             0f80feca743f4       23.5MB
docker.io/calico/node                    v3.28.0             4e42b6f329bc1       355MB
docker.io/calico/pod2daemon-flexvol      v3.28.0             587b28ecfc62e       13.4MB
registry.k8s.io/kube-proxy               v1.30.1             747097150317f       85.9MB
registry.k8s.io/pause                    3.9                 e6f1816883972       750kB
```
