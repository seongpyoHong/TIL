### Ingress vs Load Balancer
인턴 프로젝트를 진행하면서 Graylog Cluster를 Kubernetes에 배포하고, Client가 Graylog Web Interface에 접근할 수 있도록 하기 위해 Web Interface에 관련된 Service를 Cluster 외부로 노출시켜야 했다.

Service를 Cluster의 외부로 노출시키는 방법 중 Ingress와 Load Balancer가 존재하는데 두가지 요소의 명확한 차이점에 대해 정리해보고자 한다.

#### Load Balancer
Load Balancer는 하나의 service를 포워딩하여 클러스터 외부에 노출한다. 각 서비스는 자체 IP 주소를 가지게 된다.

#### Ingress
여러 서비스 앞에서 Router / Entrypoint 역할을 한다. 또한, L7 라우팅을 지원하여 Path에 따라 서비스를 구분할 수 있다.