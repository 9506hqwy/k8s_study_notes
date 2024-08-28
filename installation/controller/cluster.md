# クラスタ

## コントロールプレーンの構築

コントロールプレーンを構築する。

```sh
kubeadm init --control-plane-endpoint=controller.home.local --pod-network-cidr=192.168.0.0/16
```

```
[init] Using Kubernetes version: v1.30.1
[preflight] Running pre-flight checks
        [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
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
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.061184605s
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 44.501711497s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node controller.home.local as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node controller.home.local as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 50xk2g.pgg67ykupqov7nrx
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

  kubeadm join controller.home.local:6443 --token 50xk2g.pgg67ykupqov7nrx \
        --discovery-token-ca-cert-hash sha256:8afa26c573f015cbd81c9e320d33111fbd9d6e0a43a1cf6024976de1529c6d8b \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join controller.home.local:6443 --token 50xk2g.pgg67ykupqov7nrx \
        --discovery-token-ca-cert-hash sha256:8afa26c573f015cbd81c9e320d33111fbd9d6e0a43a1cf6024976de1529c6d8b
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

### ネットワーク名前空間

ネットワーク名前空間を確認する。

```sh
ip netns
```

```
e3e2e0a1-062b-480a-a483-660254e2abe0 (id: 0)
c1c0eace-08c5-4bdd-9420-6b215f3fefc5
cb0e0d0a-ff37-4145-9b11-aca82942493d (id: 1)
a99f9b7e-5f7c-4ec8-bf45-cb64d51456e0
e7b1d7bb-4b7d-4ea6-8afd-8b24ee745b24
322f5081-3c51-4541-a1dc-5ac30daa5713
dfb4ea86-a48b-4ee0-bf68-62892c0f3838
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
    link/ether 00:15:5d:bf:ba:60 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 0ad645ee-5652-4cb9-8a61-d70dbad17ffe
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:63 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev e64b23d6-2347-4eb9-9ab7-3274451a7178
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2e:ef:76:4d:74:2f brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  141.18 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
5: veth2c3744a3@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 76:c7:d5:bb:81:33 brd ff:ff:ff:ff:ff:ff link-netns e3e2e0a1-062b-480a-a483-660254e2abe0 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
6: vethf3cf048b@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP mode DEFAULT group default
    link/ether 4a:9e:48:be:23:c9 brd ff:ff:ff:ff:ff:ff link-netns cb0e0d0a-ff37-4145-9b11-aca82942493d promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin on guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.2e:ef:76:4d:74:2f designated_root 8000.2e:ef:76:4d:74:2f hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec e3e2e0a1-062b-480a-a483-660254e2abe0 ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 6e:ce:a9:a6:c0:4c brd ff:ff:ff:ff:ff:ff link-netns c1c0eace-08c5-4bdd-9420-6b215f3fefc5 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
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
    link/ether ae:7b:c6:41:f1:08 brd ff:ff:ff:ff:ff:ff link-netns c1c0eace-08c5-4bdd-9420-6b215f3fefc5 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
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
    link/ether 6e:ce:a9:a6:c0:4c brd ff:ff:ff:ff:ff:ff link-netns c1c0eace-08c5-4bdd-9420-6b215f3fefc5
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
    link/ether ae:7b:c6:41:f1:08 brd ff:ff:ff:ff:ff:ff link-netns c1c0eace-08c5-4bdd-9420-6b215f3fefc5
    inet 10.85.0.5/16 brd 10.85.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ac7b:c6ff:fe41:f108/64 scope link
       valid_lft forever preferred_lft forever
```

### ルート

ルーティングを確認する。

```sh
ip route show
```

```
default via 172.16.0.254 dev eth0 proto static metric 100
10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.11 metric 101
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1
172.16.0.0/24 dev eth0 proto kernel scope link src 172.16.0.11 metric 100
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

### iptables

フィルタを確認する。

```sh
iptables -n -L
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
KUBE-PROXY-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes load balancer firewall */
KUBE-NODEPORTS  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes health check service ports */
KUBE-EXTERNAL-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes externally-visible service portals */
KUBE-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
KUBE-PROXY-FIREWALL  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes load balancer firewall */
KUBE-FORWARD  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes service portals */
KUBE-EXTERNAL-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes externally-visible service portals */

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
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
```

NAT を確認する。

```sh
iptables -t nat -n -L
```

```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
KUBE-SERVICES  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
KUBE-POSTROUTING  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
CNI-0753f7d0856b570d6e276401  0    --  10.85.0.4            0.0.0.0/0            /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */
CNI-48a9f059df3e8ab377561c26  0    --  10.85.0.5            0.0.0.0/0            /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */

Chain CNI-0753f7d0856b570d6e276401 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "a0f922c5fc8fbe1a021f6a3fd300177b7334010970a661c086795c5e65f96708" */

Chain CNI-48a9f059df3e8ab377561c26 (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            10.85.0.0/16         /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */
MASQUERADE  0    --  0.0.0.0/0           !224.0.0.0/4          /* name: "crio" id: "b7a5dc968252fd5de9770f38c6a36abf44ac23779d07294a4c81babfe0ff66d2" */

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination

Chain KUBE-MARK-MASQ (11 references)
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

Chain KUBE-SEP-WAIILD3LBHUAKL5L (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  0    --  10.85.0.4            0.0.0.0/0            /* kube-system/kube-dns:dns */
DNAT       17   --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns */ udp to:10.85.0.4:53

Chain KUBE-SERVICES (2 references)
target     prot opt source               destination
KUBE-SVC-NPX46M4PTMTKRN6Y  6    --  0.0.0.0/0            10.96.0.1            /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SVC-ERIFXISQEP7F7OF4  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */ tcp dpt:53
KUBE-SVC-JD5MR3NA4I4DYORP  6    --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */ tcp dpt:9153
KUBE-SVC-TCOU7JCQXEZGVUNU  17   --  0.0.0.0/0            10.96.0.10           /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
KUBE-NODEPORTS  0    --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL

Chain KUBE-SVC-ERIFXISQEP7F7OF4 (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */ tcp dpt:53
KUBE-SEP-63Q3QMXTSXKJR2EZ  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 10.85.0.4:53 */ statistic mode random probability 0.50000000000
KUBE-SEP-A4KLBZUL4ZZDQPAH  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns-tcp -> 10.85.0.5:53 */

Chain KUBE-SVC-JD5MR3NA4I4DYORP (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */ tcp dpt:9153
KUBE-SEP-EI34DFWCIU6LMT63  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 10.85.0.4:9153 */ statistic mode random probability 0.50000000000
KUBE-SEP-KTM2MCEYUDOLTZBE  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:metrics -> 10.85.0.5:9153 */

Chain KUBE-SVC-NPX46M4PTMTKRN6Y (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  6    -- !192.168.0.0/16       10.96.0.1            /* default/kubernetes:https cluster IP */ tcp dpt:443
KUBE-SEP-23Y66C2VAJ3WDEMI  0    --  0.0.0.0/0            0.0.0.0/0            /* default/kubernetes:https -> 172.16.0.11:6443 */

Chain KUBE-SVC-TCOU7JCQXEZGVUNU (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  17   -- !192.168.0.0/16       10.96.0.10           /* kube-system/kube-dns:dns cluster IP */ udp dpt:53
KUBE-SEP-WAIILD3LBHUAKL5L  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 10.85.0.4:53 */ statistic mode random probability 0.50000000000
KUBE-SEP-HDNVQHEBMPSP33XA  0    --  0.0.0.0/0            0.0.0.0/0            /* kube-system/kube-dns:dns -> 10.85.0.5:53 */
```

### Kubernetes リソース

クラスタにあるリソースを確認する。

```sh
kubectl get all --all-namespaces
```

```
NAMESPACE     NAME                                                READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-7db6d8ff4d-p4kzt                        1/1     Running   1          68m
kube-system   pod/coredns-7db6d8ff4d-slnrj                        1/1     Running   1          68m
kube-system   pod/etcd-controller.home.local                      1/1     Running   1          69m
kube-system   pod/kube-apiserver-controller.home.local            1/1     Running   1          69m
kube-system   pod/kube-controller-manager-controller.home.local   1/1     Running   1          69m
kube-system   pod/kube-proxy-skh2j                                1/1     Running   1          68m
kube-system   pod/kube-scheduler-controller.home.local            1/1     Running   1          69m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  69m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   69m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   69m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           69m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-7db6d8ff4d   2         2         2       68m
```

ノードを確認する。

```sh
kubectl get nodes
```

```
NAME                    STATUS   ROLES           AGE   VERSION
controller.home.local   Ready    control-plane   72m   v1.30.1
```

### コンテナ

コンテナを確認する。

```sh
crictl ps -a
```

```
CONTAINER           IMAGE                                                              CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
e0b950eafce8b       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4   18 minutes ago      Running             coredns                   1                   b7a5dc968252f       coredns-7db6d8ff4d-p4kzt
45a457c47a13f       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4   18 minutes ago      Running             coredns                   1                   a0f922c5fc8fb       coredns-7db6d8ff4d-slnrj
2c8ef1733279d       747097150317f99937cabea484cff90097a2dbd79e7eb348b71dc0af879883cd   18 minutes ago      Running             kube-proxy                1                   cd4e57dfd4400       kube-proxy-skh2j
1c7d83fbe5c01       a52dc94f0a91256bde86a1c3027a16336bb8fea9304f9311987066307996f035   18 minutes ago      Running             kube-scheduler            1                   77c0488ed357a       kube-scheduler-controller.home.local
e24f682800ae7       25a1387cdab82166df829c0b70761c10e2d2afce21a7bcf9ae4e9d71fe34ef2c   18 minutes ago      Running             kube-controller-manager   1                   fff5a0bd1bad8       kube-controller-manager-controller.home.local
892abdc3f86ba       91be9408031725d89ff709fdf75a7666cedbf0d8831be4581310a879a096c71a   18 minutes ago      Running             kube-apiserver            1                   31f69e32c888a       kube-apiserver-controller.home.local
e9164e8444789       3861cfcd7c04ccac1f062788eca39487248527ef0c0cfd477a83d7691a75a899   19 minutes ago      Running             etcd                      1                   fee47cf00de95       etcd-controller.home.local
```

コンテナイメージを確認する。

```sh
crictl images
```

```
IMAGE                                     TAG                 IMAGE ID            SIZE
registry.k8s.io/coredns/coredns           v1.11.1             cbb01a7bd410d       61.2MB
registry.k8s.io/etcd                      3.5.12-0            3861cfcd7c04c       151MB
registry.k8s.io/kube-apiserver            v1.30.1             91be940803172       118MB
registry.k8s.io/kube-controller-manager   v1.30.1             25a1387cdab82       112MB
registry.k8s.io/kube-proxy                v1.30.1             747097150317f       85.9MB
registry.k8s.io/kube-scheduler            v1.30.1             a52dc94f0a912       63MB
registry.k8s.io/pause                     3.9                 e6f1816883972       750kB
```
