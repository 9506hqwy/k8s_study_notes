# コンテランタイム

## Linux ブリッジの設定

br_netfilter, overlay モジュールを有効化する。

```sh
cat > /etc/modules-load.d/k8s.conf <<EOF
br_netfilter
overlay
EOF

systemctl restart systemd-modules-load
```

フォワーディングを有効にする。

```sh
firewall-cmd --permanent --zone=public --add-masquerade
firewall-cmd --reload
```

## cri-o のインストール

インストール対象を指定する。

```sh
export KUBERNETES_VERSION=v1.30
export PROJECT_PATH=prerelease:/main
```

kubernetes パッケージをインストールするためリポジトリを有効化する。

```sh
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/repodata/repomd.xml.key
EOF
```

CRI-O パッケージをインストールするためリポジトリを有効化する。

```sh
cat > /etc/yum.repos.d/cri-o.repo <<EOF
[cri-o]
name=CRI-O
baseurl=https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/rpm/repodata/repomd.xml.key
EOF
```

コンテナのセキュリティポリシーを自動で管理するためのパッケージをインストールする。

```sh
dnf install -y container-selinux
```

インストールする。

```sh
dnf install -y cri-o kubelet kubeadm kubectl
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now crio
```

## スワップの設定

スワップを無効化する。

```sh
swapoff -a
sed -i -e '/swap/d' /etc/fstab
```
