# NTP サーバ

## 設定

内部ネットワーク向けに NTP をサービスを有効化する。
外部 NTP サーバは ntp.nict.jp を参照する。

```sh
sed \
    -e 's/^\(pool.*\)/#\1/' \
    \
    -e '/^#pool/aserver ntp.nict.jp iburst' \
    \
    -e '/^#allow/aallow 10.0.0.0/24' \
    \
    -i /etc/chrony.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=ntp
firewall-cmd --reload
```

## 起動

サービスを再起動する。

```sh
systemctl restart chronyd
```

設定を確認する。

```sh
chronyc sources
```

```
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ntp-a3.nict.go.jp             1   6     7     1   -549us[-2167us] +/-   13ms
```
