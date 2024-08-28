# 時刻同期

## 設定

NTP サーバは gateway を参照する。

```sh
sed \
    -e 's/^\(pool.*\)/#\1/' \
    \
    -e '/^#pool/aserver gateway iburst' \
    \
    -i /etc/chrony.conf
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
^? gateway.home.local            0   6     0     -     +0ns[   +0ns] +/-    0ns
```
