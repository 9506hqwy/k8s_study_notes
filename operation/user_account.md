# ユーザアカウント

Kubernetes はユーザアカウントのリソースは管理していない。

## ユーザに権限付与

ユーザ `developer` に権限を付与する。

```sh
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
```

```text
role.rbac.authorization.k8s.io/developer created
```

```sh
kubectl create rolebinding developer-admin --role=developer --user=developer
```

```text
rolebinding.rbac.authorization.k8s.io/developer-admin created
```

## サービスアカウントに権限付与

サービスアカウントに権限を付与する。

```sh
kubectl create serviceaccount developer-sa
```

```text
serviceaccount/developer-sa created
```

```sh
kubectl create rolebinding developer-sa-admin --role=developer --serviceaccount=default:developer-sa
```

```text
rolebinding.rbac.authorization.k8s.io/developer-sa-admin created
```

## クライアント証明書

クライアント証明書を使用して認証を行う。

クライアントの秘密鍵を作成する。

```sh
openssl genrsa -out client.key 4096
```

ユーザ `developer`, グループ `dev` の証明書署名要求を作成する。

```sh
openssl req -new -key client.key -out client.csr -subj "/CN=developer/O=dev"
```

証明書署名要求を登録する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: developer
spec:
  request: $(cat client.csr | base64 -w 0)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

```text
certificatesigningrequest.certificates.k8s.io/developer created
```

証明書要求を確認する。

```sh
kubectl get csr
```

```text
NAME        AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
developer   44s   kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              Pending
```

証明書要求を承認する。

```sh
kubectl certificate approve developer
```

```text
certificatesigningrequest.certificates.k8s.io/developer approved
```

証明書要求を確認する。

```sh
kubectl get csr
```

```text
NAME        AGE    SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
developer   3m5s   kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              Approved,Issued
```

クライアント証明書を取得する。

```sh
kubectl get csr developer -o jsonpath='{.status.certificate}' | base64 -d > client.crt
```

kubeconfig に認証情報を追加する。

```sh
kubectl config set-credentials developer --client-certificate=client.crt --client-key=client.key --embed-certs=true
```

```text
User "developer" set.
```

kubeconfig にコンテキストを追加する。

```sh
kubectl config set-context developer --cluster=kubernetes --user=developer
```

```text
Context "developer" created.
```

コンテキストを切り替える。

```sh
kubectl config use-context developer
```

```text
Switched to context "developer".
```

接続確認する。

```sh
kubectl get pods
```

```text
No resources found in default namespace.
```

## サービスアカウントトークン

JSON Web Token を生成する。

```sh
kubectl create token developer-sa
```

```text
eyJhbGciOiJSUzI...
```

kubeconfig に認証情報を追加する。

```sh
kubectl config set-credentials developer-sa --token=$TOKEN
```

```text
User "developer-sa" set.
```

kubeconfig にコンテキストを追加する。

```sh
kubectl config set-context developer-sa --cluster=kubernetes --user=developer-sa
```

```text
Context "developer-sa" created.
```

コンテキストを切り替える。

```sh
kubectl config use-context developer-sa
```

```text
Switched to context "developer-sa".
```

接続確認する。

```sh
kubectl get pods
```

```text
No resources found in default namespace.
```

## 参考

- [Certificates and Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
