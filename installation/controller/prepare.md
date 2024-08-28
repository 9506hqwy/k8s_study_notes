# 事前準備

## ネットワーク

ホスト名を設定する。

```sh
hostnamectl set-hostname controller.home.local
```

ネットワークアダプタを設定する。

```sh
nmcli connection modify eth1 \
    ipv6.method disabled \
    ipv4.method manual \
    ipv4.addresses 10.0.0.11/24 \
    connection.autoconnect yes
nmcli connection up eth1

nmcli connection modify eth0 \
    ipv6.method disabled \
    ipv4.method manual \
    ipv4.dns 172.16.0.254 \
    ipv4.dns-search home.local \
    ipv4.addresses 172.16.0.11/24 \
    ipv4.gateway 172.16.0.254 \
    connection.autoconnect yes
nmcli connection up eth0
```

ファイアウォールを設定する。

```sh
firewall-cmd --set-default-zone trusted
firewall-cmd --permanent --zone=public --change-interface=eth0
firewall-cmd --permanent --zone=internal --change-interface=eth1
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

## NTP クライアント

[](../appendix/time_sync.md) を参照する。

## コンテナランタイム

[](../appendix/container_runtime.md) を参照する。

## ファイアウォールの設定

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=6443/tcp
firewall-cmd --permanent --zone=public --add-port=2379-2380/tcp
firewall-cmd --reload
```
