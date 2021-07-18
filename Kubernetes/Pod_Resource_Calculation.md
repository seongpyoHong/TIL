### Pod Resource Calculation

최근 Resource Quota가 제한된 환경에서 K8S를 사용하면서 Pod의 Resource가 어떻게 계산되는지에 대해서 헷갈렸던 부분과 새롭게 알게된 부분에 대해서 정리해보고자 한다.

> 참고) Resource Quota를 초과하는 경우(request, limit 등),Pod의 생성이 불가하게 된다.

- effective init request/limit : Init Container들의 리소스의 request/limit 중 가장 큰 값
- Pod's effective request/limit : Init Container를 제외한 Pod의 모든 컨테이너의 request/limit의 합 vs effective init request/limit 중 큰 값
- QoS Class는 init container와 app container 모두에게 동일하게 적용


### 참고 자료
- https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#resources
