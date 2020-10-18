### Headless Service

#### Serivce란?

Kubernetes에서 Service는 클러스터 내의 Pods을 클러스터 외부로 노출시키는 역할은 한다. 

Pods은 임시적이기 떄문에 IP의 변경이 발생한다. Pods의 IP 변경에도 Client가 동일한 IP 주소로 접속하기 위해서 Service에 할당되는 ClusterIP는 불변성을 가지며, 이 주소를 통해 Pods에 접근할 수 있다.

Service는 Load Balancing / Proxy 기능을 가지기 때문에 Service의 ClusterIP로 접근할 경우, Service에 해당하는 Pods들이 상황에 맞게 랜덤으로 반환된다.



#### Headless Service

DB와 같이 Pod의 재시작에도 상태가 유지되는 어플리케이션을 구축하기 위해, Pod의 상태 정보가 유지되는 StatefulSet을 사용한다. 이 때, StatefulSet으로 배포되는 어플리케이션은 자체적으로 Load Balancing 기능을 가지고 있기 때문에 Service에서 제공되는 Load Balancing 기능이 필요 없다. 또한, Service에 요청을 보내서 랜덤하게 Pod을 받는 기존의 Service와 달리 특정 Pods의 주소를 알아야 하는 경우가 존재한다. 이를 커버하기 위한 메커니즘이 **Headless Service** 이다.

Headless Service에 요청을 보내면 해당 Service에 속하는 모든 Pods의 IP 주소를 전부 반환하며, 이 Service에 속하는 Pods 간에 DNS를 통하여 직접 요청이 가능하다.