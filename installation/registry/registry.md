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
curl -sSL https://github.com/goharbor/harbor/releases/download/v1.10.19/harbor-online-installer-v1.10.19.tgz \
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

Note: docker-compose version: 2.29.6


[Step 2]: preparing environment ...

[Step 3]: preparing harbor configs ...
prepare base dir is set to /root/harbor
/usr/src/app/utils/configs.py:100: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  configs = yaml.load(f)
/usr/src/app/utils/configs.py:90: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  versions = yaml.load(f)
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /secret/keys/secretkey
Generated certificate, key file: /secret/core/private_key.pem, cert file: /secret/registry/root.crt
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

WARN[0000] /root/harbor/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion


[Step 4]: starting Harbor ...
WARN[0000] /root/harbor/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 59/33
 ? jobservice Pulled                                                                                                                                                                               10.1s
 ? proxy Pulled                                                                                                                                                                                    67.7s
 ? core Pulled                                                                                                                                                                                     67.7s
 ? registryctl Pulled                                                                                                                                                                              20.0s
 ? portal Pulled                                                                                                                                                                                    7.2s
 ? postgresql Pulled                                                                                                                                                                               67.7s
 ? log Pulled                                                                                                                                                                                      67.7s
 ? redis Pulled                                                                                                                                                                                    19.8s
 ? registry Pulled                                                                                                                                                                                 24.5s
[+] Running 10/10
 ? Network harbor_harbor        Created                                                                                                                                                             0.2s
 ? Container harbor-log         Started                                                                                                                                                             0.6s
 ? Container harbor-portal      Started                                                                                                                                                             1.1s
 ? Container harbor-db          Started                                                                                                                                                             1.2s
 ? Container registry           Started                                                                                                                                                             1.1s
 ? Container registryctl        Started                                                                                                                                                             1.1s
 ? Container redis              Started                                                                                                                                                             1.1s
 ? Container harbor-core        Started                                                                                                                                                             1.5s
 ? Container nginx              Started                                                                                                                                                             2.0s
 ? Container harbor-jobservice  Started                                                                                                                                                             2.0s
? ----Harbor has been installed and started successfully.----
```

コンテナが起動していることを確認する。

```sh
docker ps
```

```
CONTAINER ID   IMAGE                                  COMMAND                   CREATED         STATUS                   PORTS                                         NAMES
09b054522640   goharbor/harbor-jobservice:v1.10.19    "/harbor/harbor_jobs…"   4 minutes ago   Up 4 minutes (healthy)                                                 harbor-jobservice
f2d7f7907bc9   goharbor/nginx-photon:v1.10.19         "nginx -g 'daemon of…"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp   nginx
7f83c4ed0a77   goharbor/harbor-core:v1.10.19          "/harbor/harbor_core"     4 minutes ago   Up 4 minutes (healthy)                                                 harbor-core
a8b1c743e748   goharbor/harbor-portal:v1.10.19        "nginx -g 'daemon of…"   4 minutes ago   Up 4 minutes (healthy)   8080/tcp                                      harbor-portal
2e6eb172ee84   goharbor/harbor-registryctl:v1.10.19   "/home/harbor/start.…"   4 minutes ago   Up 4 minutes (healthy)                                                 registryctl
4fbe4ba38548   goharbor/registry-photon:v1.10.19      "/home/harbor/entryp…"   4 minutes ago   Up 4 minutes (healthy)   5000/tcp                                      registry
de967baac6c6   goharbor/redis-photon:v1.10.19         "redis-server /etc/r…"   4 minutes ago   Up 4 minutes (healthy)   6379/tcp                                      redis
6a8d77a6986d   goharbor/harbor-db:v1.10.19            "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes (healthy)   5432/tcp                                      harbor-db
6738e057da26   goharbor/harbor-log:v1.10.19           "/bin/sh -c /usr/loc…"   4 minutes ago   Up 4 minutes (healthy)   127.0.0.1:1514->10514/tcp                     harbor-log
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

イメージを作成するときはマニフェスト形式に `docker` を指定する。

```sh
buildah bud --format=docker ....
```
