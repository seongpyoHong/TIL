Istio를 설치하는 과정에서 CRD라는 커스텀 리소스를 위한 옵션을 알게되었다. 관련 설명을 읽던 중 Operator, Controller, API Server, Controller Manager와 같은 용어들이 등장했고 실제로 쿠버네티스를 사용하면서 내부 구조에 대한 공부를 해본적이 없는 것 같아 이번 기회에 정리해보고자 한다.

### Component

먼저, 쿠버네티스의 구조는 어플리케이션을 실행하는 **워커 노드**와 이를 관리하는 **컨트롤 플레인**으로 이루어진다.
[https://kubernetes.io/ko/docs/concepts/overview/components/](https://kubernetes.io/ko/docs/concepts/overview/components/)

### Control Plane

먼저 컨트롤 플레인부터 살펴보면 API server, etcd, controller manager, scheduler로 이루어져 있다.

- API server

    쿠버네티스의 모든 작업은 API를 통해서 이루어지며, API를 이용한 요청은 API Server를 통해 이루어진다. (Control Plane의 Frontend)

- etcd

    분산 key/value 저장소로 클러스터에서 사용되는 상태, 정보들을 저장

- Scheduler

    Pod을 배치할 Node를 선정

- controller-manager

    전체 컨트롤러를 구동 ex) Node Controller, SA Controller 등

### **Worker Node**

worker node는 kubelet, kube-proxy, cAdvisor, pod으로 이루어져 있다.

- kubelet

    Pod spec을 통해 컨테이너가 Pod 내에서 목표한대로 동작하도록 관리한다.

- kube-proxy

    네크워크 프록시로 Service의 구현부이다. kube-proxy를 통해 클러스터의 내/외부와 통신 한다.

- cAdvisor

    동작 중인 컨테이너의 메트릭을 모니터링하고 이를 API Server로 전송한다.

- Pod

    어플리케이션이 실행되는 요소로 쿠버네티스에서 가장 작은 배포 단위를 의미한다.

### 참고 자료

[https://kubernetes.io/ko/docs/concepts/overview/components/](https://kubernetes.io/ko/docs/concepts/overview/components/)

 [https://honggg0801.tistory.com/42](https://honggg0801.tistory.com/42)
