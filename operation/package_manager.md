# パッケージマネージャ

パッケージマネージャとして Helm を使用する。

動作確認に [k8s-api-server-rs/chart](https://github.com/9506hqwy/k8s-api-server-rs) を使用する。

## パッケージのインストール

名前 `sample` でローカルのディレクトリにあるパッケージをインストールする。

```sh
helm install sample . --set imagePath=registry.home.local/system/sample-api-server
```

```
NAME: sample
LAST DEPLOYED: Fri Nov  8 20:11:28 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

パッケージを確認する。

```sh
helm list
```

```
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
sample  default         1               2024-11-08 20:11:28.654848096 +0900 JST deployed        k8s-sample-api-server-0.2.0
```

## パッケージの作成

```{note}
RSA / ECDSA でも署名に下記のエラーで失敗する。

Error: openpgp: unsupported feature: public key type: 22

- [helm package --sign throws "Error: private key not found"](https://github.com/helm/helm/issues/7631)
- [Error: openpgp: unsupported feature: public key type: 22](https://github.com/helm/helm/issues/11634)
```

パッケージを作成する。

```sh
helm package .
```

```
Successfully packaged chart and saved it to: /root/projects/k8s-api-server-rs/chart/k8s-sample-api-server-0.2.0.tgz
```

パッケージをインストールする。

```sh
helm install sample k8s-sample-api-server-0.2.0.tgz --set imagePath=registry.home.local/system/sample-api-server
```

```
NAME: sample
LAST DEPLOYED: Fri Nov  8 22:23:52 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## パッケージのレジストリ登録

レジストリにログインする。

```sh
helm registry login registry.home.local --insecure
```

```
Username: admin
Password:
Login Succeeded
```

レジストリに登録する。

```sh
helm push k8s-sample-api-server-0.2.0.tgz oci://registry.home.local/library --insecure-skip-tls-verify
```

```
Pushed: registry.home.local/library/k8s-sample-api-server:0.2.0
Digest: sha256:6a2707c089c1a4418544a52ef9a64f67125ba4b1665e878a3b16ddb5f8fd0dcf
```

パッケージをインストールする。

```sh
helm install sample oci://registry.home.local/library/k8s-sample-api-server \
    --version 0.2.0 \
    --set imagePath=registry.home.local/system/sample-api-server \
    --insecure-skip-tls-verify
```

```
Pulled: registry.home.local/library/k8s-sample-api-server:0.2.0
Digest: sha256:6a2707c089c1a4418544a52ef9a64f67125ba4b1665e878a3b16ddb5f8fd0dcf
NAME: sample
LAST DEPLOYED: Sat Nov  9 00:47:19 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## リポジトリの追加

```{note}
OCI レジストリはリポジトリを登録できない。

- [helm repo add library https://core.harbor.domain/library Error](https://github.com/goharbor/harbor/issues/18720)
```

## パッケージの削除

```sh
helm uninstall sample
```

```
release "sample" uninstalled
```

## Chart

### ビルドインオブジェクト

テンプレート内で使用するコンテキストは以下の構成となる。

```yaml
Release:
  Name: :string <リリースの名前>
  Namespace: :string <名前空間名>
  IsUpgrade: :bool <upgrade?>
  IsInstall: :bool <install?>
Values: :object <values.yaml>
Chart: :object <Chart.yaml>
Subcharts: :object <charts/*>
Files: <ファイルの操作>
Capabilities: :object <Kubernetes の情報>
Tamplate:
  Name: :string <ファイルパス>
  BasePath: :string <ディレクトリパス>
```

#### Release

`Release` オブジェクトを使用する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Release.Name }}
  label:
    namespace: {{ .Release.Namespace }}
    is-upgrade: {{ .Release.IsUpgrade }}
    is-install: {{ .Release.IsInstall }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: chart-1731113987
  label:
    namespace: default
    is-upgrade: false
    is-install: true
```

#### Chart

`Chart` オブジェクトを使用する。
ビルトインオブジェクトのプロパティは UpperCamelCase になる。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    type: {{ .Chart.Type }}
    version: {{ .Chart.Version }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    type: application
    version: 0.1.0
```

#### Capabilities

`Capabilities` オブジェクトを使用する。
文字列はダブルクォートで囲む。文字列はシングルクォートで囲むとエラーになる。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    apiVersions: {{ .Capabilities.APIVersions | first }}
    apiVersionsHas: {{ .Capabilities.APIVersions.Has "v1" }}
    kubeVersion: {{ .Capabilities.KubeVersion.Major }}.{{ .Capabilities.KubeVersion.Minor }}
    helmVersion: {{ .Capabilities.HelmVersion.Version }}-{{ .Capabilities.HelmVersion.GitCommit }}
```

リリース名を自動生成して出力する。
リストの順序は保証されていないため `.Capabilities.APIVersions` の結果は実行ごとに変化する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    apiVersions: projectcalico.org/v3/IPReservation
    apiVersionsHas: true
    kubeVersion: 1.31
    helmVersion: v3.16.2-13654a52f7c70a143b1dd51416d633e1071faffb
```

#### Tamplte

`Template` オブジェクトを使用する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    name: {{ .Template.Name }}
    basePath: {{ .Template.BasePath }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    name: helm-chart/templates/sample.yaml
    basePath: helm-chart/templates
```

### 制御構文

#### 条件分岐

`if-else` で条件分岐する。
false, 0, 空文字、空集合が偽と判定される。

`if-else-end` の文は空行になるため `{{-` で空行を削除する。
`{{-` は前の空白(改行を含む)を削除する。
`-}}` は後の空白(改行を含む)を削除する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- if .Values.cond }}
    cond: true
    {{- else }}
    cond: false
    {{- end }}
```

真の判定を出力する。

```sh
helm install --dry-run --generate-name . --set cond=true
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    cond: true
```

偽の判定を出力する。

```sh
helm install --dry-run --generate-name .
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    cond: false
```

#### ループ

`range` でループする。
ブロック内はスコープが指定した値となる。ルートオブジェクトを参照する場合は `$` を使用する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- range .Values.names }}
    - value: {{ $.Chart.Name }}-{{ . }}
    {{- end }}
```

配列を指定して出力する。

```sh
helm install --dry-run --generate-name . --set-json 'names=["a", "b"]'
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - value: helm-chart-a
    - value: helm-chart-b
```

#### スコープ

`with` でブロック内のオブジェクト参照を制限する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- with .Capabilities.KubeVersion }}
    - kubeVersrion: {{ .Major }}.{{ .Minor }}
    {{- end }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - kubeVersrion: 1.31
```

### 名前付きテンプレート

ファイル内にテンプレートを定義する。
呼び出し時にオブジェクトを指定するとスコープを制限する。

```yaml
{{- define "helm-chart.kube" }}
    - kubeVersrion: {{ .Major }}.{{ .Minor }}
{{- end }}
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- template "helm-chart.kube" .Capabilities.KubeVersion }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - kubeVersrion: 1.31
```

パイプラインにテンプレートの結果を渡すには `include` を使用する。

```yaml
{{- define "helm-chart.kube" }}
- kubeVersrion: {{ .Major }}.{{ .Minor }}
{{- end }}
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- include "helm-chart.kube" .Capabilities.KubeVersion | indent 4 }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - kubeVersrion: 1.31
```

名前付きテンプレートを別ファイルに定義する。
ファイル名は `_` で始める。`_` で始めると Kubernetes のマニフェストではないと判断される。
定義したテンプレートは、他のファイルのどこからでも参照できる。

```yaml
# templates/_helpers.tpl
{{- define "helm-chart.kube" }}
- kubeVersrion: {{ .Major }}.{{ .Minor }}
{{- end }}
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- include "helm-chart.kube" .Capabilities.KubeVersion | indent 4 }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - kubeVersrion: 1.31
```

### 変数の定義

`$xxxx` で変数の定義と参照ができる。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- $value := "1.31" }}
    - kubeVersrion: {{ $value }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - kubeVersrion: 1.31
```

### ファイルの参照

外部ファイルを取得する。
*templates*, *charts* や *.helmignore* に指定されたファイルは対象外になる。

```ini
# file.toml
[section]
key = value
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    {{- range $file, $_ := .Files }}
    - {{ $file }}
    {{- end }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    - .helmignore
    - file.toml
```

ファイルの内容を取得する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    file: |-
      {{- range .Files.Lines "file.toml" }}
      {{ . }}
      {{- end }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    file: |-
      [section]
      key = value
```

ファイル名と内容をマップで出力する。内容を base64 でエンコードする場合は `.AsSecrets` を使用する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
{{ (.Files.Glob "*.toml").AsConfig | indent 4 }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    file.toml: |-
      [section]
      key = value
```

ファイルの内容を base64 でエンコードする。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    file: {{ .Files.Get "file.toml" | b64enc }}
```

リリース名を自動生成して出力する。

```sh
helm install --dry-run --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    file: W3NlY3Rpb25dCmtleSA9IHZhbHVl
```

### リソースの取得

既存のリソースを使用する。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Chart.Name }}
  label:
    pods:
    {{- range $index, $pod := (lookup "v1" "Pod" "kube-system" "").items }}
    - {{ $pod.metadata.name  }}
    {{- end }}
```

リリース名を自動生成して出力する。API サーバに接続するため `--dry-run=server` を指定する。

```sh
helm install --dry-run=server --generate-name .
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-chart
  label:
    pods:
    - coredns-7c65d6cfc9-sr5mk
    - coredns-7c65d6cfc9-zczmd
    - etcd-controller.home.local
    - kube-apiserver-controller.home.local
    - kube-controller-manager-controller.home.local
    - kube-proxy-65fgv
    - kube-proxy-6r696
    - kube-proxy-rd46g
    - kube-scheduler-controller.home.local
```

## 参考
- [Helm - The package manager for Kubernetes](https://helm.sh/)
