# クラスタ

## コントロールプレーンの構築

コントロールプレーンを構築する。

```sh
kubeadm init --control-plane-endpoint=controller.home.local --pod-network-cidr=172.17.0.0/16
```

```text
I1012 15:29:03.222680    6078 version.go:261] remote version is much newer: v1.34.1; falling back to: stable-1.33
[init] Using Kubernetes version: v1.33.5
[preflight] Running pre-flight checks
        [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controller.home.local kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.0.11]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controller.home.local localhost] and IPs [172.16.0.11 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controller.home.local localhost] and IPs [172.16.0.11 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.971361ms
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://172.16.0.11:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-controller-manager is healthy after 1.450607677s
[control-plane-check] kube-scheduler is healthy after 1.646982387s
[control-plane-check] kube-apiserver is healthy after 3.001850182s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node controller.home.local as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node controller.home.local as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: g47ok9.5byuxmpdmzfi714s
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join controller.home.local:6443 --token g47ok9.5byuxmpdmzfi714s \
        --discovery-token-ca-cert-hash sha256:84aff1be3e53133b56e6bbb9f7583ff3c32dde97a4fd89bfb3beff9e5babf071 \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join controller.home.local:6443 --token g47ok9.5byuxmpdmzfi714s \
        --discovery-token-ca-cert-hash sha256:84aff1be3e53133b56e6bbb9f7583ff3c32dde97a4fd89bfb3beff9e5babf071
```

マシン起動時の自動起動を設定する。

```sh
systemctl enable kubelet
```

クラスタの接続設定をコピーする。

```sh
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

## 環境確認

Controller Node でネットワーク構成を確認する。

![Controller ノードの構成](../../_static/image/controller_cluster.png "Controller ノードの構成")

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```text
744103f0-ba89-4c6d-894c-afe0f15f1981
eb9c9108-ee52-4f8e-92c6-dbd6bb8b9377
dbcd298c-3140-4234-9a94-bcd4fb650efd
d6d8d075-35c1-4ec2-8965-91aeb0e30fe2
ecfaad58-f4b1-4439-8a54-e36d80a89c46 (id: 0)
7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9 (id: 1)
148017df-888d-4c83-aabb-d956b1575525
```

### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:49:41:b8 brd ff:ff:ff:ff:ff:ff promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536 parentbus virtio parentdev virtio1
    altname enx5254004941b8
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:91:a0:96 brd ff:ff:ff:ff:ff:ff promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535 addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536 parentbus virtio parentdev virtio2
    altname enx52540091a096
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether ea:d0:47:27:0e:9c brd ff:ff:ff:ff:ff:ff promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.ea:d0:47:27:e:9c designated_root 8000.ea:d0:47:27:e:9c root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  218.22 fdb_n_learned 2 fdb_max_learned 0 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mst_enabled 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
5: veth6081ed49@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether 5a:62:27:85:7f:ea brd ff:ff:ff:ff:ff:ff link-netns ecfaad58-f4b1-4439-8a54-e36d80a89c46 promiscuity 1 allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.ea:d0:47:27:e:9c designated_root 8000.ea:d0:47:27:e:9c hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off neigh_vlan_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
6: veth7d4e981d@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default qlen 1000
    link/ether b6:f2:93:8b:1e:c4 brd ff:ff:ff:ff:ff:ff link-netns 7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9 promiscuity 1 allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.ea:d0:47:27:e:9c designated_root 8000.ea:d0:47:27:e:9c hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off neigh_vlan_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec ecfaad58-f4b1-4439-8a54-e36d80a89c46 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 22:61:d0:f4:11:c7 brd ff:ff:ff:ff:ff:ff link-netns 744103f0-ba89-4c6d-894c-afe0f15f1981 promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec 7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
2: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether c6:18:c6:2c:e9:25 brd ff:ff:ff:ff:ff:ff link-netns 744103f0-ba89-4c6d-894c-afe0f15f1981 promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 4 numrxqueues 4 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536
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
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:49:41:b8 brd ff:ff:ff:ff:ff:ff
    altname enx5254004941b8
    inet 172.16.0.11/24 brd 172.16.0.255 scope global noprefixroute enp1s0
       valid_lft forever preferred_lft forever
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:91:a0:96 brd ff:ff:ff:ff:ff:ff
    altname enx52540091a096
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute enp2s0
       valid_lft forever preferred_lft forever
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ea:d0:47:27:0e:9c brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.1/16 brd 10.85.255.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 1100:200::1/24 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::e8d0:47ff:fe27:e9c/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
5: veth6081ed49@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default qlen 1000
    link/ether 5a:62:27:85:7f:ea brd ff:ff:ff:ff:ff:ff link-netns ecfaad58-f4b1-4439-8a54-e36d80a89c46
    inet6 fe80::5862:27ff:fe85:7fea/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
6: veth7d4e981d@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default qlen 1000
    link/ether b6:f2:93:8b:1e:c4 brd ff:ff:ff:ff:ff:ff link-netns 7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9
    inet6 fe80::b4f2:93ff:fe8b:1ec4/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec ecfaad58-f4b1-4439-8a54-e36d80a89c46 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 22:61:d0:f4:11:c7 brd ff:ff:ff:ff:ff:ff link-netns 744103f0-ba89-4c6d-894c-afe0f15f1981
    inet 10.85.0.3/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 1100:200::3/24 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::2061:d0ff:fef4:11c7/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec 7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host proto kernel_lo
       valid_lft forever preferred_lft forever
2: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether c6:18:c6:2c:e9:25 brd ff:ff:ff:ff:ff:ff link-netns 744103f0-ba89-4c6d-894c-afe0f15f1981
    inet 10.85.0.2/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 1100:200::2/24 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::c418:c6ff:fe2c:e925/64 scope link proto kernel_ll
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
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev enp1s0 proto kernel scope link src 172.16.0.11 metric 100
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec ecfaad58-f4b1-4439-8a54-e36d80a89c46 ip route show
```

```text
default via 10.85.0.1 dev eth0
10.85.0.0/16 dev eth0 proto kernel scope link src 10.85.0.3
```

ネットワーク名前空間内のルーティングを確認する。

```sh
ip netns exec 7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9 ip route show
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
table ip mangle {
        chain KUBE-IPTABLES-HINT {
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain KUBE-FIREWALL {
                ip saddr != 127.0.0.0/8 ip daddr 127.0.0.0/8  ct status dnat counter packets 0 bytes 0 drop
        }

        chain OUTPUT {
                type filter hook output priority filter; policy accept;
                ct state new  counter packets 2679 bytes 161772 jump KUBE-PROXY-FIREWALL
                ct state new  counter packets 2679 bytes 161772 jump KUBE-SERVICES
                counter packets 162499 bytes 33185711 jump KUBE-FIREWALL
        }

        chain INPUT {
                type filter hook input priority filter; policy accept;
                ct state new  counter packets 2280 bytes 136800 jump KUBE-PROXY-FIREWALL
                 counter packets 115321 bytes 21987340 jump KUBE-NODEPORTS
                ct state new  counter packets 2280 bytes 136800 jump KUBE-EXTERNAL-SERVICES
                counter packets 162330 bytes 33139405 jump KUBE-FIREWALL
        }

        chain KUBE-KUBELET-CANARY {
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-EXTERNAL-SERVICES {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                ct state new  counter packets 50 bytes 3050 jump KUBE-PROXY-FIREWALL
                 counter packets 52 bytes 3370 jump KUBE-FORWARD
                ct state new  counter packets 50 bytes 3050 jump KUBE-SERVICES
                ct state new  counter packets 50 bytes 3050 jump KUBE-EXTERNAL-SERVICES
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
}
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain KUBE-KUBELET-CANARY {
        }

        chain CNI-7634124192b06586a22435ca {
                ip daddr 10.85.0.0/16  counter packets 0 bytes 0 accept
                ip daddr != 224.0.0.0/4  counter packets 4 bytes 265 masquerade
        }

        chain CNI-325fa1ea8df1df6fe641bff7 {
                ip daddr 10.85.0.0/16  counter packets 0 bytes 0 accept
                ip daddr != 224.0.0.0/4  counter packets 4 bytes 265 masquerade
        }

        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                 counter packets 2614 bytes 157456 jump KUBE-POSTROUTING
                ip saddr 10.85.0.3  counter packets 4 bytes 265 jump CNI-325fa1ea8df1df6fe641bff7
                ip saddr 10.85.0.2  counter packets 4 bytes 265 jump CNI-7634124192b06586a22435ca
        }

        chain KUBE-PROXY-CANARY {
        }

        chain KUBE-SERVICES {
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-ERIFXISQEP7F7OF4
                meta l4proto tcp ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-SVC-JD5MR3NA4I4DYORP
                meta l4proto tcp ip daddr 10.96.0.1  tcp dport 443 counter packets 0 bytes 0 jump KUBE-SVC-NPX46M4PTMTKRN6Y
                meta l4proto udp ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-SVC-TCOU7JCQXEZGVUNU
                 fib daddr type local counter packets 2168 bytes 130080 jump KUBE-NODEPORTS
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                 counter packets 2612 bytes 157286 jump KUBE-SERVICES
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                 counter packets 4 bytes 290 jump KUBE-SERVICES
        }

        chain KUBE-POSTROUTING {
                meta mark & 0x00004000 != 0x00004000 counter packets 2486 bytes 149718 return
                counter packets 0 bytes 0 meta mark set mark xor 0x4000
                 counter packets 0 bytes 0 masquerade fully-random
        }

        chain KUBE-NODEPORTS {
        }

        chain KUBE-MARK-MASQ {
                counter packets 0 bytes 0 meta mark set mark or 0x4000
        }

        chain KUBE-SVC-NPX46M4PTMTKRN6Y {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.1  tcp dport 443 counter packets 2 bytes 120 jump KUBE-MARK-MASQ
                 counter packets 2 bytes 120 jump KUBE-SEP-23Y66C2VAJ3WDEMI
        }

        chain KUBE-SEP-23Y66C2VAJ3WDEMI {
                ip saddr 172.16.0.11  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 2 bytes 120 dnat to 172.16.0.11:6443
        }

        chain KUBE-SVC-ERIFXISQEP7F7OF4 {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-BCWYETF6BYLYKE2H
                 counter packets 0 bytes 0 jump KUBE-SEP-HMFIVQIFQZV23PVT
        }

        chain KUBE-SEP-BCWYETF6BYLYKE2H {
                ip saddr 10.85.0.2  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 10.85.0.2:53
        }

        chain KUBE-SVC-JD5MR3NA4I4DYORP {
                meta l4proto tcp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  tcp dport 9153 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-55JPH7H2U4SL23HX
                 counter packets 0 bytes 0 jump KUBE-SEP-GNLGM655BVXRF66F
        }

        chain KUBE-SEP-55JPH7H2U4SL23HX {
                ip saddr 10.85.0.2  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 10.85.0.2:9153
        }

        chain KUBE-SVC-TCOU7JCQXEZGVUNU {
                meta l4proto udp ip saddr != 172.17.0.0/16 ip daddr 10.96.0.10  udp dport 53 counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                 meta random & 2147483647 < 1073741824 counter packets 0 bytes 0 jump KUBE-SEP-7SDR2WXD6EFFJZ42
                 counter packets 0 bytes 0 jump KUBE-SEP-YQTUGCSRGBQ7MJXA
        }

        chain KUBE-SEP-7SDR2WXD6EFFJZ42 {
                ip saddr 10.85.0.2  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp  meta l4proto udp counter packets 0 bytes 0 dnat to 10.85.0.2:53
        }

        chain KUBE-SEP-HMFIVQIFQZV23PVT {
                ip saddr 10.85.0.3  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 10.85.0.3:53
        }

        chain KUBE-SEP-GNLGM655BVXRF66F {
                ip saddr 10.85.0.3  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto tcp  meta l4proto tcp counter packets 0 bytes 0 dnat to 10.85.0.3:9153
        }

        chain KUBE-SEP-YQTUGCSRGBQ7MJXA {
                ip saddr 10.85.0.3  counter packets 0 bytes 0 jump KUBE-MARK-MASQ
                meta l4proto udp  meta l4proto udp counter packets 0 bytes 0 dnat to 10.85.0.3:53
        }
}
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all --all-namespaces -o wide
```

```text
NAMESPACE     NAME                                                READY   STATUS    RESTARTS   AGE    IP            NODE                    NOMINATED NODE   READINESS GATES
kube-system   pod/coredns-674b8bbfcf-csznq                        1/1     Running   0          119s   10.85.0.3     controller.home.local   <none>           <none>
kube-system   pod/coredns-674b8bbfcf-ng4p7                        1/1     Running   0          119s   10.85.0.2     controller.home.local   <none>           <none>
kube-system   pod/etcd-controller.home.local                      1/1     Running   1          2m7s   172.16.0.11   controller.home.local   <none>           <none>
kube-system   pod/kube-apiserver-controller.home.local            1/1     Running   1          2m7s   172.16.0.11   controller.home.local   <none>           <none>
kube-system   pod/kube-controller-manager-controller.home.local   1/1     Running   1          2m7s   172.16.0.11   controller.home.local   <none>           <none>
kube-system   pod/kube-proxy-xn5ft                                1/1     Running   0          119s   172.16.0.11   controller.home.local   <none>           <none>
kube-system   pod/kube-scheduler-controller.home.local            1/1     Running   1          2m8s   172.16.0.11   controller.home.local   <none>           <none>

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE    SELECTOR
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  2m8s   <none>
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   2m7s   k8s-app=kube-dns

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE    CONTAINERS   IMAGES                               SELECTOR
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   2m7s   kube-proxy   registry.k8s.io/kube-proxy:v1.33.5   k8s-app=kube-proxy

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES                                    SELECTOR
kube-system   deployment.apps/coredns   2/2     2            2           2m7s   coredns      registry.k8s.io/coredns/coredns:v1.12.0   k8s-app=kube-dns

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                    SELECTOR
kube-system   replicaset.apps/coredns-674b8bbfcf   2         2         2       2m    coredns      registry.k8s.io/coredns/coredns:v1.12.0   k8s-app=kube-dns,pod-template-hash=674b8bbfcf
```

ノードを確認する。

```sh
kubectl get nodes
```

```text
NAME                    STATUS   ROLES           AGE     VERSION
controller.home.local   Ready    control-plane   2m32s   v1.33.5
```

### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```text
CONTAINER           IMAGE                                                              CREATED             STATE               NAME                      ATTEMPT             POD ID              POD                                             NAMESPACE
4c6e898ae7237       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b   2 minutes ago       Running             coredns                   0                   cfe7a259687c1       coredns-674b8bbfcf-csznq                        kube-system
3a54b420f13f8       2844ee7bb56c2c194e1f4adafb9e7b60b9ed16aa4d07ab8ad1f019362e2efab3   2 minutes ago       Running             kube-proxy                0                   af7d36903da84       kube-proxy-xn5ft                                kube-system
8691f15fdee1b       1cf5f116067c67da67f97bff78c4bbc76913f59057c18627b96facaced73ea0b   2 minutes ago       Running             coredns                   0                   91788731bfe42       coredns-674b8bbfcf-ng4p7                        kube-system
bf1301e4a02b5       b7335a56022aba291f5df653c01b7ab98d64fb5cab221378617f4a1236e06a62   3 minutes ago       Running             kube-apiserver            1                   0fac08d8108f1       kube-apiserver-controller.home.local            kube-system
a91c74eb5a7f8       499038711c0816eda03a1ad96a8eb0440c005baa6949698223c6176b7f5077e1   3 minutes ago       Running             etcd                      1                   c8313f7ac8e3c       etcd-controller.home.local                      kube-system
50e12d9f285a1       8bb43160a0df4d7d34c89d9edbc48735bc2f830771e4b501937338221be0f668   3 minutes ago       Running             kube-controller-manager   1                   5b77f18b14ce8       kube-controller-manager-controller.home.local   kube-system
d733221b081b3       33b680aadf474b7e5e73957fc00c6af86dd0484c699c8461ba33ee656d1823bf   3 minutes ago       Running             kube-scheduler            1                   a5cfc43153f06       kube-scheduler-controller.home.local            kube-system
```

kube-proxy が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 3a54b420f13f8 | jq '.info.pid')
```

```text
148017df-888d-4c83-aabb-d956b1575525
```

coredns が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 4c6e898ae7237 | jq '.info.pid')
```

```text
ecfaad58-f4b1-4439-8a54-e36d80a89c46
```

```sh
ip netns identify $(crictl inspect 8691f15fdee1b | jq '.info.pid')
```

```text
7b76c567-0934-43fb-a3fa-ba1fd6e9c0f9
```

etcd が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect a91c74eb5a7f8 | jq '.info.pid')
```

```text
148017df-888d-4c83-aabb-d956b1575525
```

kube-scheduler が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect d733221b081b3 | jq '.info.pid')
```

```text
148017df-888d-4c83-aabb-d956b1575525
```

kube-apiserver が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect bf1301e4a02b5 | jq '.info.pid')
```

```text
148017df-888d-4c83-aabb-d956b1575525
```

kube-controller-manager が属するネットワーク名前空間を確認する。

```sh
ip netns identify $(crictl inspect 50e12d9f285a1 | jq '.info.pid')
```

```text
148017df-888d-4c83-aabb-d956b1575525
```

コンテナイメージを確認する。

```sh
crictl images
```

```text
IMAGE                                     TAG                 IMAGE ID            SIZE
registry.k8s.io/coredns/coredns           v1.12.0             1cf5f116067c6       71.2MB
registry.k8s.io/etcd                      3.5.21-0            499038711c081       154MB
registry.k8s.io/kube-apiserver            v1.33.5             b7335a56022ab       103MB
registry.k8s.io/kube-controller-manager   v1.33.5             8bb43160a0df4       95.8MB
registry.k8s.io/kube-proxy                v1.33.5             2844ee7bb56c2       99.3MB
registry.k8s.io/kube-scheduler            v1.33.5             33b680aadf474       74.6MB
registry.k8s.io/pause                     3.10                873ed75102791       742kB
```
