# コンテナレジストリ

Harbor を構築する。

## Docker インストール

リポジトリを追加する。

```sh
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
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

```
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
```

## Docker Compose インストール

```sh
curl -sSL https://github.com/docker/compose/releases/download/v2.29.6/docker-compose-linux-x86_64 \
    -o docker-compose \
    --output-dir /usr/local/bin
chmod +x /usr/local/bin/docker-compose
```

## 自己署名証明書の作成

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

```
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

## Harbor インストール

インストーラをダウンロードする。

```sh
curl -sSL https://github.com/goharbor/harbor/releases/download/v2.12.0/harbor-online-installer-v2.12.0.tgz \
    -o harbor-online-installer.tgz
tar -zxf harbor-online-installer.tgz
```

パラメータファイル *harboar.yaml* をもとに *harboar.yml* を作成する。
下記以外は既定値のままとする。

- `hostname`: FQDN
- `https.certificate`: サーバ証明書の絶対パス
- `https.private_key`: サーバ秘密鍵の絶対パス

インストールする。

```sh
./install.sh
```

```
[Step 0]: checking if docker is installed ...

Note: docker version: 27.3.1

[Step 1]: checking docker-compose is installed ...

Note: Docker Compose version v2.29.7


[Step 2]: preparing environment ...

[Step 3]: preparing harbor configs ...
prepare base dir is set to /root/harbor
Unable to find image 'goharbor/prepare:v2.12.0' locally
v2.12.0: Pulling from goharbor/prepare
69f3f4f936bb: Pull complete
653c57bbe511: Pull complete
d54cebaef548: Pull complete
078852956263: Pull complete
af8b1169f99d: Pull complete
2f5c2336c48c: Pull complete
64720024dfda: Pull complete
3fe468d45fa2: Pull complete
dbc6d12bbf4c: Pull complete
b9ee61559eb8: Pull complete
Digest: sha256:8c3be33b8ecc4226bd29e958d7a0f1eb160fe2db1addaf598cc37306a5eaf748
Status: Downloaded newer image for goharbor/prepare:v2.12.0
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
loaded secret from file: /data/secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir


Note: stopping existing Harbor instance ...


[Step 4]: starting Harbor ...
[+] Running 62/46
 ? postgresql Pulled                                                               23.4s
 ? registry Pulled                                                                 53.0s
 ? core Pulled                                                                     56.2s
 ? portal Pulled                                                                   27.8s
 ? log Pulled                                                                      53.0s
 ? proxy Pulled                                                                    19.5s
 ? redis Pulled                                                                     9.1s
 ? registryctl Pulled                                                              53.0s
 ? jobservice Pulled                                                               53.0s
[+] Running 10/10
 ? Network harbor_harbor        Created                                             0.2s
 ? Container harbor-log         Started                                             0.5s
 ? Container registryctl        Started                                             1.1s
 ? Container registry           Started                                             1.0s
 ? Container harbor-db          Started                                             0.8s
 ? Container redis              Started                                             1.1s
 ? Container harbor-portal      Started                                             1.1s
 ? Container harbor-core        Started                                             1.5s
 ? Container harbor-jobservice  Started                                             2.0s
 ? Container nginx              Started                                             2.0s
? ----Harbor has been installed and started successfully.----
```

コンテナが起動していることを確認する。

```sh
docker ps
```

```
CONTAINER ID   IMAGE                                 COMMAND                   CREATED          STATUS                    PORTS                                         NAMES
3d83411ce1f9   goharbor/harbor-jobservice:v2.12.0    "/harbor/entrypoint.…"   29 minutes ago   Up 28 minutes (healthy)                                                 harbor-jobservice
e6e2681c10bb   goharbor/nginx-photon:v2.12.0         "nginx -g 'daemon of…"   29 minutes ago   Up 29 minutes (healthy)   0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp   nginx
3fac8045fd96   goharbor/harbor-core:v2.12.0          "/harbor/entrypoint.…"   29 minutes ago   Up 29 minutes (healthy)                                                 harbor-core
1259e9ccad40   goharbor/harbor-registryctl:v2.12.0   "/home/harbor/start.…"   29 minutes ago   Up 29 minutes (healthy)                                                 registryctl
4d10dc72f4fb   goharbor/harbor-portal:v2.12.0        "nginx -g 'daemon of…"   29 minutes ago   Up 29 minutes (healthy)                                                 harbor-portal
3e895874ab38   goharbor/harbor-db:v2.12.0            "/docker-entrypoint.…"   29 minutes ago   Up 29 minutes (healthy)                                                 harbor-db
4f60d1159bb4   goharbor/redis-photon:v2.12.0         "redis-server /etc/r…"   29 minutes ago   Up 29 minutes (healthy)                                                 redis
71c5225e8854   goharbor/registry-photon:v2.12.0      "/home/harbor/entryp…"   29 minutes ago   Up 29 minutes (healthy)                                                 registry
9d8a91523e59   goharbor/harbor-log:v2.12.0           "/bin/sh -c /usr/loc…"   29 minutes ago   Up 29 minutes (healthy)   127.0.0.1:1514->10514/tcp                     harbor-log
```

ブラウザでアクセスして `admin` でログインする。

## 動作確認

事前にブラウザでログインしてプロジェクトを作成する。

レジストリにログインする。

```sh
docker login registry.home.local
```

```
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded
```

docker.io から pull して Harbor に push する。

```sh
docker pull hello-world
docker tag hello-world registry.home.local/test/hello-world
docker push registry.home.local/test/hello-world
```

```
Using default tag: latest
The push refers to repository [registry.home.local/test/hello-world]
ac28800ec8bb: Pushed
latest: digest: sha256:d37ada95d47ad12224c205a938129df7a3e52345828b4fa27b03a98825d1e2e7 size: 524
```

## コンテナランタイム

ワーカノードに CA 証明書を配置する。

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
