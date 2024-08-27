# 事前準備

## ネットワーク

ホスト名を設定する。

```sh
hostnamectl set-hostname gateway.home.local
```

ネットワークアダプタを設定する。

```sh
nmcli connection modify eth1 \
    ipv6.method disabled \
    ipv4.method manual \
    ipv4.dns-search home.local \
    ipv4.addresses 172.16.0.254/24 \
    connection.autoconnect yes
nmcli connection up eth1

nmcli connection modify eth0 \
    ipv6.method disabled \
    ipv4.method auto \
    connection.autoconnect yes
nmcli connection up eth0
```

ファイアウォールを設定する。

```sh
firewall-cmd --permanent --zone=external --change-interface=eth0
firewall-cmd --permanent --zone=internal --change-interface=eth1

firewall-cmd --permanent --zone=external --add-masquerade
firewall-cmd --permanent --zone=internal --add-forward

firewall-cmd --permanent --new-policy forwarding
firewall-cmd --permanent --policy forwarding --add-ingress-zone internal
firewall-cmd --permanent --policy forwarding --add-egress-zone external
firewall-cmd --permanent --policy forwarding --set-priority 100
firewall-cmd --permanent --policy forwarding --set-target ACCEPT

firewall-cmd --reload
```

IPv6 を無効化する。

```sh
cat > /etc/sysctl.d/10-disable-ipv6.conf <<EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
EOF
```

再起動する。
