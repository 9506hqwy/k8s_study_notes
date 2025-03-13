# ストレージ

コンテナストレージとして iSCSI を構築する。

## イニシエータのインストール

ワーカノードに iSCSI イニシエータをインストールする。

```sh
dnf install -y iscsi-initiator-utils
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now iscsid
```

イニシエーター名を確認する。

```sh
cat /etc/iscsi/initiatorname.iscsi
```

```text
InitiatorName=iqn.1994-05.com.redhat:6c13bfc4535e
```

## ターゲットのインストール

ストレージサーバに iSCSI ターゲットをインストールする。

```sh
dnf install -y lvm2 targetcli
```

## LUN の作成

*/dev/vdb* を物理ボリュームとして作成する。

```sh
pvcreate /dev/vdb
```

```text
  Physical volume "/dev/vdb" successfully created.
  Creating devices file /etc/lvm/devices/system.devices
```

物理ボリュームから LVM ボリュームグループを作成する。

```sh
vgcreate container-volumes /dev/vdb
```

```text
  Volume group "container-volumes" successfully created
```

LVM ボリュームグループから LUN を作成する。

```sh
lvcreate -L 1G container-volumes -n disk01
```

```text
  Logical volume "disk01" created.
```

LUN を確認する。

```sh
lvdisplay
```

```text
  --- Logical volume ---
  LV Path                /dev/container-volumes/disk01
  LV Name                disk01
  VG Name                container-volumes
  LV UUID                DgGZXI-jnlr-bHcd-bH7u-e1Mt-V3zt-qXIpUR
  LV Write Access        read/write
  LV Creation host, time storage.home.local, 2024-10-13 11:40:37 +0900
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
```

## ファイアウォールの設定

iSCSI ターゲットを許可する。

```sh
firewall-cmd --permanent --zone=internal --add-service=iscsi-target
firewall-cmd --reload
```

## iSCSI ターゲットの起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now target
```

起動を確認する。

```sh
targetcli ls
```

```text
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 0]
  o- loopback ......................................................................................................... [Targets: 0]
```

## ターゲットの作成

iSCSI ターゲットを作成する。

```sh
targetcli /iscsi create
```

```text
Created target iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
```

iSCSI ターゲットを確認する。

```sh
targetcli ls /iscsi/iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31/
```

```text
o- iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31 ............................................................. [TPGs: 1]
  o- tpg1 ................................................................................................... [no-gen-acls, no-auth]
    o- acls .............................................................................................................. [ACLs: 0]
    o- luns .............................................................................................................. [LUNs: 0]
    o- portals ........................................................................................................ [Portals: 1]
      o- 0.0.0.0:3260 ......................................................................................................... [OK]
```

## バックストアの作成

iSCSI ターゲットに LUN を追加する。

```sh
targetcli /backstores/block create name=disk01 dev=/dev/container-volumes/disk01
```

```text
Created block storage object disk01 using /dev/container-volumes/disk01.
```

バックストアを確認する。

```sh
targetcli ls /backstores/block/disk01
```

```text
o- disk01 .......................................................... [/dev/container-volumes/disk01 (1.0GiB) write-thru deactivated]
  o- alua ......................................................................................................... [ALUA Groups: 1]
    o- default_tg_pt_gp ............................................................................. [ALUA state: Active/optimized]
```

## ターゲットにバックストアを追加

iSCSI ターゲットにバックストアを追加する。

```sh
targetcli /iscsi/iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31/tpg1/luns/ create /backstores/block/disk01
```

```text
Created LUN 0.
```

iSCSI LUN を確認する。

```sh
targetcli ls /iscsi/iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31/tpg1/luns
```

```text
o- luns .................................................................................................................. [LUNs: 1]
  o- lun0 ........................................................ [block/disk01 (/dev/container-volumes/disk01) (default_tg_pt_gp)]
```

## ACL の作成

iSCSI ACL を作成する。

```sh
targetcli /iscsi/iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31/tpg1/acls create iqn.1994-05.com.redhat:6c13bfc4535e
```

```text
Created Node ACL for iqn.1994-05.com.redhat:6c13bfc4535e
Created mapped LUN 0.
```

iSCSI ACL を確認する。

```sh
targetcli ls /iscsi/iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31/tpg1/acls/iqn.1994-05.com.redhat:6c13bfc4535e
```

```text
- iqn.1994-05.com.redhat:6c13bfc4535e ............................................................................ [Mapped LUNs: 1]
  o- mapped_lun0 .......................................................................................... [lun0 block/disk01 (rw)]
```

## 設定の保存

設定を永続化する。

```sh
targetcli saveconfig
```

```text
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json
```
