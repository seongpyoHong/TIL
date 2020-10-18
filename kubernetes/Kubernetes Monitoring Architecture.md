쿠버네티스에서 수집하는 메트릭은 크게 2개의 메트릭으로 분류된다. 

- **System Metric**

    노드, 컨테이너의 CPU,Memory 사용량과 같은 시스템 관련 메트릭을 의미한다. 

- **Service Metric**

    어플리케이션을 모니터링 하는데 필요한 메트릭을 의미한다. 서비스 메트릭은 컨테이너에서 나오는 메트릭과 어플리케이션에서 발생하는 메트릭으로 나눌 수 있다. 서비스 메트릭은 custom 메트릭으로 HPA에서 사용될 수 있다.

쿠버네티스의 모니터링 구조는 크게 2가지의 파이프라인으로 구분된다. 

- **Core Metric Pipeline**

    핵심 요소들에 대한 모니터링을 담당한다. kubelet에 내장된 cAdvisor에서 수집해서 메모리에 가지고 있으며, api-server에서는 이를 내보낼 수 있는 Metrics API를 제공한다. 

    **Metrics-Server**

    Metrics Server는 kubelet로 부터 메트릭 데이터를 수집하고, api-server에 이를 Metrics API로 노출한다. 이 API를 HPA 및 `kubectl top` 에서 사용할 수 있다.

    공식 문서를 참고하면 Metrics Server는 HPA가 아닌 목적, 예를 들면 모니터링 솔루션에 데이터 전달 및 모니터링 솔루션의 데이터 소스로 사용하는 것을 권장하지 않고 이를 위해서는 외부 모니터링 솔루션 (Prometheus etc.)을 사용하는 것을 권장한다.

    또한, Metrics Server로 수집되는 데이터를 CPU, Memory와 같은 제한적인 데이터이기 때문에 다른 메트릭의 수집이 필요하다면 외부 모니터링 솔루션 사용이 필요하다.

- **Monitoring Pipeline**

    쿠버네티스 외의 컴포넌트를 이용하는 모니터링 솔루션으로 Prometheus를 주로 사용한다.

    모니터링 파이프라인에서 수집되는 데이터는 다음과 같은 종류가 있다.

    - core / non-core metric
    - application metric
    - kubernetes infra container metric

    수집된 커스텀 메트릭을 HPA에 사용하기 위해서는 API Adapter를 생성해서 custom metric API를 노출해야한다. Core Metric Pipeline을 통해 수집된 Metrics API는 `/apis/metrics.k8s.io` 을 통해, Custom Metric은 `/apis/custom.metrics.k8s.io` 을 통해 노출된다.

### 참고 자료

- [https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md#architecture](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md#architecture)
- [https://arisu1000.tistory.com/27855?category=787056](https://arisu1000.tistory.com/27855?category=787056)
- [https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md#architecture](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md#architecture)
