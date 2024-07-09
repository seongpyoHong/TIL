## LimitRange

Namespace 별로 리소스의 사용과 생성을 제한할 수 있다. LimitRange는 요청할 수 있는 `request`, `limit`의 min, max 값을 조정하거나 resource를 명시하지 않았을 때 default로 설정되는 request를 정의할 수 있다.

> 내부적으로는 LimitRanger Admission Controller가 LimitRange에 정의된 리소스들의 사용량을 추적하며, 만약 제약 조건을 위반했을 경우에는 `403 Forbidden` 요청에 따라 API 서버에 대한 요청이 실패한다.

1. Example (default limit & request) : 아래 예제에서는 CPU에 대해 default limit을 `1` 로 default request를 `0.5` 로 설정한다.

    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-limit-range
    spec:
      limits:
      - default:
          cpu: 1
        defaultRequest:
          cpu: 0.5
        type: Container
    ```
    문서에서는 여러가지 조건에 대해서 나와있다.
    - limit O, request X : request = limit으로 설정 ([ref](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/#what-if-you-specify-a-container-s-limit-but-not-its-request))
    - limit X, request X : limit은 namespace default로 설정 (cpu : `1` , memory : `512Mi`)

2. Example (min & max) : 아래 예제에서는 memory request에 대한 min/max 값을 제한한다. 이를 만족하지 않는 경우 Pod의 생성이 제한된다.

    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: mem-min-max-demo-lr
    spec:
      limits:
      - max:
          memory: 1Gi
        min:
          memory: 500Mi
        type: Container
    ```

**만약 compute resource에 대한 min/max를 지정하지 않으면 어떻게 될까?** 

⇒ limitRange에 의해 정의된 request, limit default 값을 따라간다.

---
## ResourceQuota

LimitRange가 개별 리소스에 대한 min,max 및 default 값을 정의했다면 ResourceQuota는 네임스페이스 전체의 리소스 사용량이 이를 초과하지 않도록 제한한다. 

문서를 참고하면 Compute/Storage/Object Count Quota 등이 존재한다.

- [compute-resource-quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/#compute-resource-quota)
- [storage-resource-quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/#storage-resource-quota)
- [object-count-quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/#object-count-quota)

### Quota Scope

ResourceQuota에 대하여 유효성 검사를 수행하기 위한 리소스의 범위를 제한할 수 있다.

제공하는 범위는 문서 참고 : [quota-scopes](https://kubernetes.io/docs/concepts/policy/resource-quotas/#quota-scopes)


> ResourceQuota 및 LimitRange 모두 admission controller를 통해 동작한다. 따라서 API server에서 `admission-control` 옵션은 통해서 활성화해야한다.
