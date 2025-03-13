# カスタムリソース定義

Kubernetes のカスタムリソースを定義する仕組み。

## カスタムリソース定義の登録

カスタムリソース定義を作成する。

`metadata.name` は `spec.names.plural`.`spec.group` とする必要がある。
`spec.versions.name.schema.openAPIV3Schema` も必須になる。

```yaml
# custom-sample-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: samples.custom.sample.crd
spec:
  group: custom.sample.crd
  names:
    plural: samples
    singular: sample
    kind: Sample
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
```

```sh
kubectl apply -f custom-sample-crd.yaml
```

```text
customresourcedefinition.apiextensions.k8s.io/samples.custom.sample.crd created
```

カスタムリソース定義を確認する。

```sh
kubectl get crd samples.custom.sample.crd
```

```text
NAME                        CREATED AT
samples.custom.sample.crd   2024-10-26T01:21:26Z
```

## カスタムリソースの作成

カスタムリソースを作成する。

```yaml
# custom-sample-cr-01.yaml
apiVersion: custom.sample.crd/v1alpha1
kind: Sample
metadata:
  name: custom-sample-cr-01
```

```sh
kubectl apply -f custom-sample-cr-01.yaml
```

```text
sample.custom.sample.crd/custom-sample-cr-01 created
```

カスタムリソースを確認する。

```sh
kubectl get sample custom-sample-cr-01
```

```text
NAME                  AGE
custom-sample-cr-01   81s
```

## 検証

入力に対する値を検証する方法として以下がある。

- OpenAPIV3Schema
- [Common Expression Language](https://github.com/google/cel-spec)
- [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Validating Admission Policy](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)

```{note}
TODO
```

## 参考

- [CustomResourceDefinition](https://kubernetes.io/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/)
- [JSON Schema Specification](https://json-schema.org/specification)
