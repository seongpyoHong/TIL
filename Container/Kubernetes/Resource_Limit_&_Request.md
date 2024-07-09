최근 인프라 직무 면접 때 Kubernetes상에서 리소스 관리에 대한 질문을 받았는데, 대답을 못한 부분이 있어서 이번 기회에 다시 한 번 정리해보고자 한다.
쿠버네티스는 Pod에 대한 스케줄링 지표로 컨테이너에 필요한 리소스의 양을 명시하여 사용할 수 있다. 현재 지원하고 있는 리소스 타입은 CPU와 Memory이다.

### 단위

- CPU

ms를 사용하며 이는 해당 컨테이너에서 얼마만큼의 CPU 자원을 할당할 것인가를 정의한다. (조대협님의 글에 따르면 대략 1000ms가 1 vCore라고 한다.) 

참고 자료 [조대협님의 블로그](https://bcho.tistory.com/1291)

- Memory

메모리의 단위는 MB를 사용한다.

### Request & Limit
쿠버네티스에서 컨테이너에 적용될 리소스의 양을 위한 옵션으로 `request`와 `limit` 이 존재한다.

- request

컨테이너가 생성될 때 요청하는 양

- limit

컨테이너가 실행되다가 리소스가 더 필요한 경우 요청할 수 있는 최대 할당량

### ResourceQuote & LimitRange

쿠버네티스에서는 네임스페이스 및 컨테이너 별로 사용할 수 있는 리소스의 양을 정할 수 있는 기능을 제공한다.

- Resource Quota

네임스페이스 별로 사용할 수 있는 리소스의 양

- LimitRange

컨테이너 개별 자원의 사용 가능 범위

### Overcommitted

request에 맞게 리소스를 할당하여 컨테이너를 생성, 운영하던 도중 리소스가 더 필요하게 되면 limit까지 리소스를 요청할 수 있다.

하지만 모든 컨테이너의 limit 총합이 실제 시스템이 가용가능한 reousrce의 양보다 많아지는 현상 (=Overcommitted)이 발생할 수 있다.

Overcommitted 상태가 되었을 경우 다음과 같은 현상이 발생한다.

- CPU Overcommitted

실제 CPU 사용량을 request에 정의된 상태까지 낮춘다. (Throttling) 이후애도 Overcommitted 상태가 해결되지 않는다면 우선 순위에 따라 운영 중인 컨테이너를 강제 종료 시킨다.

- Memory Overcommitted

사용중인 메모리의 크기를 줄일 수 없기 때문에 우선 순위에 따라 컨테이너를 강제 종료 시킨다. 이후 초기 request에 맞게 컨테이너가 재할당된다.

이런 이유로 인해 컨테이너 자원에 대한 리소스를 세밀하게 조정하지 않는 경우에는 Overcommitted 현상을 막기 위해 request와 limit을 동일하게 맞추는 것을 권장한다. (만약 컨테이너가 resource limit을 가지고 있지만 request를 정의하지 않는 경우, limit과 일치하는request를 자동으로 할당한다.)

### Pod에 대한 QoS 구성

쿠버네티스가 Pod을 생성할 때 다음의 QoS 클래스 중 하나를 할당한다.

- Guaranteed

Pod에 Guaranteed QoS 클래스 할당을 위한 전제 조건은 다음과 같다.

1. Pod의 모든 컨테이너는 memory/cpu에 대해 동일한 `request` , `limit`을 가지고 있어야 한다.
- Burstable

다음의 경우 Pod에 Burstable QoS 클래스가 부여된다.

1. Pod이 Guranateed Qos 클래스 기준을 만족하지 않는다.
2. Pod 내부에서 최소한 하나의 컨테이너가 리소스에 대한 `request`를 가진다.
- BestEffort

다음의 경우 Pod에 BestEffort QoS 클래스가 부여된다.

Pod내부의 컨테이너에 대해 리소스 `request`, `limit` 을 가지지 않는다.

**Global OOM Killer**

현재 컨테이너가 `limit` 보다 적게 메모리를 사용하고 있어도 Burstable, Best Effort클래스의 Pod으로 인해 OOM을 감지할 수 있다. (다른 컨테이너가 limit만큼 사용하고 있기 때문에)

쿠버네티스의 노드에 대해 적용되는 OOM은 다음과 같이 2가지가 존재한다.

- Kubelet Level

kubelet OOM Killer는 Allocatable이라는 노드에서 사용 가능한 리소스의 크기를 정의한 값에 맞춰 OOM을 유발한 Pod을 Eviction API를 통해 종료시킨다.

이 때, 우선순위에 따라 종료된 Pod을 결정하게 된다. 

참고: [https://m.blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=221676471427&referrerCode=0&searchKeyword=request](https://m.blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=221676471427&referrerCode=0&searchKeyword=request) 님의 글을 보게 되면 우선순위를 메기는 방법은 

1. 사용량이 request를 초과한 Pod 선택
2. 메모리 사용량으로 정렬

하는 방법을 사용한다. (Guranteed Qos Class는 request를 넘길 일이 없기 때문에 OOM Killer에 의해 종료되지 않는다.)

- OS Level

kubelet이 메모리 상태를 감지하기 전에 노드의 메모리 사용량이 초과된다면 리눅스 자체의 OOM Killer가 작동할 수 있다. 이 때도 우선순위에 의해 Pod 내부 프로세스가 종료된다.

- Guranteed Class : -998
- kubelet, docker daemon : -999
- BestEffort Class : 1000
- Burstable Class : request가 낮을 수록 높은 값을 가진다.

Kubelet Level의 OOM Killer는 Pod 내부의 모든 프로세스를 종료시키지만, OS Level의 OOM Killer가 동작할시 컨테이너 내부의 특정 프로세스만 강제 종료 시킨다. 이로 인해 init process가 아닌 child process가 종료되고 이로인해 정상 동작하는 것처럼 보일 수도 있기 때문에 child process의 종료를 init process에 전달하는 이미지 설계가 필요하다.
