# コンテナレジストリ

Harbor を構築する。

## Docker インストール

リポジトリを追加する。

```sh
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```text
repo の追加: https://download.docker.com/linux/centos/docker-ce.repo
```

Docker をインストールする。

```sh
dnf install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin; \
```

Docker を起動する。

```sh
systemctl enable --now docker
```

```text
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
```

## 自己署名証明書の作成

openssl をインストールする。

```sh
dnf install -y openssl
```

CA 証明書の秘密鍵を作成する。

```sh
openssl genrsa -out ca.key 4096
```

CA 証明書を作成する。

```sh
openssl req -x509 -new -sha512 \
    -days 3650 \
    -subj "/C=JP/ST=State/L=Location/O=Home/OU=Personal/CN=Root CA" \
    -key ca.key \
    -out ca.crt
```

サーバの秘密鍵を作成する。

```sh
openssl genrsa -out registry.home.local.key 4096
```

サーバの証明書署名要求を作成する。

```sh
openssl req -new -sha512 \
    -subj "/C=JP/ST=State/L=Location/O=Home/OU=Personal/CN=registry.home.local" \
    -key registry.home.local.key \
    -out registry.home.local.csr
```

x509 v3 拡張ファイルを作成する。

```sh
cat >v3.ext <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=registry.home.local
DNS.2=registry
EOF
```

サーバの証明書を作成する。

```sh
openssl x509 -req -sha512 \
    -days 3650 \
    -extfile v3.ext \
    -CA ca.crt \
    -CAkey ca.key \
    -CAcreateserial \
    -in registry.home.local.csr \
    -out registry.home.local.crt
```

```text
Certificate request self-signature ok
subject=C=JP, ST=State, L=Location, O=Home, OU=Personal, CN=registry.home.local
```

ファイルの拡張子を変更する。

```sh
openssl x509 -inform PEM -in registry.home.local.crt -out registry.home.local.cert
```

Docker に証明書を設定する。

```sh
mkdir -p /etc/docker/certs.d/registry.home.local/
cp registry.home.local.cert /etc/docker/certs.d/registry.home.local/
cp registry.home.local.key /etc/docker/certs.d/registry.home.local/
cp ca.crt /etc/docker/certs.d/registry.home.local/
```

Docker を再起動する。

```sh
systemctl restart docker
```

## Harbor インストール

インストーラをダウンロードする。

```sh
HARBOR_VERSION=2.14.0
curl -sSL "https://github.com/goharbor/harbor/releases/download/v${HARBOR_VERSION}/harbor-online-installer-v${HARBOR_VERSION}.tgz" \
    -o harbor-online-installer.tgz
tar -zxf harbor-online-installer.tgz
```

パラメータファイル *harbor.yml.tmpl* をもとに *harboar.yml* を作成する。
下記以外は既定値のままとする。

- `hostname`: FQDN
- `https.certificate`: サーバ証明書の絶対パス
- `https.private_key`: サーバ秘密鍵の絶対パス

インストールする。

```sh
./prepare
```

```text
prepare base dir is set to /root/harbor
Unable to find image 'goharbor/prepare:v2.14.0' locally
v2.14.0: Pulling from goharbor/prepare
02edd478168e: Pull complete
cfdadf0c5f3d: Pull complete
1b28ecd864ab: Pull complete
403f4f1d5670: Pull complete
36ddfdd25509: Pull complete
5b1565560a7a: Pull complete
470b5fa53727: Pull complete
39bea911ca37: Pull complete
7b9b4146531c: Pull complete
4f08a495e85c: Pull complete
Digest: sha256:23f9bcbc86e60336a424d1607b61f5ed97a8402828c3aba5319bf774480da586
Status: Downloaded newer image for goharbor/prepare:v2.14.0
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
copy /data/secret/tls/harbor_internal_ca.crt to shared trust ca dir as name harbor_internal_ca.crt ...
ca file /hostfs/data/secret/tls/harbor_internal_ca.crt is not exist
copy  to shared trust ca dir as name storage_ca_bundle.crt ...
copy None to shared trust ca dir as name redis_tls_ca.crt ...
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
```

コンテナを起動する。

```sh
docker compose up -d
```

コンテナが起動していることを確認する。

```sh
docker ps
```

```text
CONTAINER ID   IMAGE                                 COMMAND                   CREATED              STATUS                        PORTS                                         NAMES
37846473102b   goharbor/harbor-jobservice:v2.14.0    "/harbor/entrypoint.…"   About a minute ago   Up About a minute (healthy)                                                 harbor-jobservice
b8dc7c4d78d8   goharbor/nginx-photon:v2.14.0         "nginx -g 'daemon of…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp   nginx
8c87302c85c1   goharbor/harbor-core:v2.14.0          "/harbor/entrypoint.…"   About a minute ago   Up About a minute (healthy)                                                 harbor-core
345f48447ab8   goharbor/harbor-db:v2.14.0            "/docker-entrypoint.…"   About a minute ago   Up About a minute (healthy)                                                 harbor-db
063af68598d3   goharbor/harbor-registryctl:v2.14.0   "/home/harbor/start.…"   About a minute ago   Up About a minute (healthy)                                                 registryctl
94e7a2510ba6   goharbor/harbor-portal:v2.14.0        "nginx -g 'daemon of…"   About a minute ago   Up About a minute (healthy)                                                 harbor-portal
85e5f5fe470c   goharbor/redis-photon:v2.14.0         "redis-server /etc/r…"   About a minute ago   Up About a minute (healthy)                                                 redis
40a91335036d   goharbor/registry-photon:v2.14.0      "/home/harbor/entryp…"   About a minute ago   Up About a minute (healthy)                                                 registry
c4a102cfe8b8   goharbor/harbor-log:v2.14.0           "/bin/sh -c /usr/loc…"   About a minute ago   Up About a minute (healthy)   127.0.0.1:1514->10514/tcp                     harbor-log
```

ブラウザでアクセスして `admin` でログインする。

## 動作確認

事前にブラウザでログインしてプロジェクトを作成する。

レジストリにログインする。

```sh
docker login registry.home.local
```

```text
Username: admin
Password:

WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

docker.io から pull して Harbor に push する。

```sh
docker pull hello-world
docker tag hello-world registry.home.local/test/hello-world
docker push registry.home.local/test/hello-world
```

```text
Using default tag: latest
The push refers to repository [registry.home.local/test/hello-world]
53d204b3dc5d: Pushed
latest: digest: sha256:19459a6bbefb63f83f137f08c1df645f8846e2cd1f44fe209294ebc505e6495e size: 524
```

## コンテナランタイム

コントローラノード、ワーカノードに CA 証明書を配置する。

```sh
mkdir -p /etc/containers/certs.d/registry.home.local
cp ca.crt /etc/containers/certs.d/registry.home.local/
```

## Podman

Podman で HTTP を使用してコンテナレジストリに接続する。

```sh
cat <<EOF >/etc/containers/registries.conf.d/100-registry.home.local.conf
[[registry]]
location="registry.home.local"
insecure=true
EOF
```

## Harbor の自動起動

マシンの起動時に Harbor の自動起動を設定する。

```sh
cat > /etc/systemd/system/harbor.service <<EOF
[Unit]
Description=Harbor Container Registry
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/bin/docker compose up
ExecStop=/bin/kill -INT ${MAINPID}
Restart=always
WorkingDirectory=/root/harbor

[Install]
WantedBy=multi-user.target
EOF
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now harbor
```

```text
Created symlink '/etc/systemd/system/multi-user.target.wants/harbor.service' → '/etc/systemd/system/harbor.service'.
```

## Harbor アンインストール

コンテナを削除する。

```sh
docker-compose down -v
```

データベースを削除する。

```sh
rm -rf /data/database
rm -rf /data/registry
rm -rf /data/redis
```
