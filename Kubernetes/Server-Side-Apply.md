User(or Controller)가 리소스를 선언적인 설정을 통해 관리할 수 있도록 해주는 feature이며, 새로운 object merge algorithm이다.

kubectl apply로 구현된 client-side apply 기능을 server-side로 대체하여 `kubectl` 이 아닌 다른 클라이언트에 의해서도 해당 로직이 사용될 수 있도록 한다.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
  namespace: default
  labels:
    test-label: test
  managedFields:
  - manager: kubectl
    operation: Apply
    apiVersion: v1
    time: "2010-10-10T0:00:00Z"
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          f:test-label: {}
      f:data:
        f:key: {}
data:
  key: some value
```

`metadata.managedFields`는 해당 리소스가 어떤 operation과 APi version을 통해 관리되는지에 대한 정보를 포함하고 있다.

> API Server에 의해 관리됨. 유저는 수정 불가


### 왜 생겼나?
- client-side dry-run은 API server 자체와 통신하지 않는다. 따라서 admission controller의 영향을 확인할 수 없다. SSA dry-run은 API server 위에서 동작하며 admission controller를 거치기 때문에 실제 영향을 파악하기 쉽다. (etcd에는 저장되지 않음)
- field ownership을 명확하게 함으로써 수정에 대한 제약을 걸 수 있다.

### Reference
- https://kubernetes.io/docs/reference/using-api/server-side-apply/
- https://opensource.googleblog.com/2021/10/server-side-apply-in-kubernetes.html
- https://medium.com/swlh/break-down-kubernetes-server-side-apply-5d59f6a14e26

