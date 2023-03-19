### Reference
- https://coffeewhale.com/k8s/network/2019/05/11/k8s-network-02/
- https://coffeewhale.com/k8s/network/2019/05/30/k8s-network-03/
- https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture
- https://kubernetes.io/ko/docs/reference/command-line-tools-reference/kube-proxy/

### Service Network
Pod은 Kubernetes에서 언제든 대체될 수 있기 때문에 Pod이 재생성되었을 때 이전 IP 주소와 동일함을 보장하지 못한다.

#### Service
Pod으로 트래픽을 포워딩해주는 Proxy 역할을 수행하는 리소스
- 어떤 Pod으로 트래픽을 포워딩할 지는 `selector` 를 통해서 결정
- Service가 생성되면 IP가 부여
- Kubernetes에서는 클러스터 내부에서 사용할 수 있는 Service용 DNS를 제공

##### 어떻게 Service IP가 할당되는가?
자체 구축 시 `kube-apiserver`에서 `service-cluster-ip-range` 값에 해당하는 IP 대역폭을 사용
- kubelet 에서 `-pod-cidr` 값으로도 확인 가능
- Pod의 IP는 host에서 bridge interface를 확인하면 실제로 device가 존재한다. (가상 ethernet interface들이 Pod 끼리 통신하기 위해 bridge에 연결된 device)
- Service는 host에서 `ifconfig` 조회해도 관련된 device가 나오지 않는다. + gateway의 routing table을 확인해도 Service 네트워크에 대한 라우팅 정보 X

#### Service IP Flow
<img width="473" alt="image" src="https://user-images.githubusercontent.com/44525736/226187242-a253d5a0-3422-466c-bea0-253343a3382c.png">
- Background
	- IP 네트워크는 자신의 host에서 해당 IP에 대한 주소를 찾지 못하면 상위 gateway로 전달한다.
- Service IP 대역은 eth0에서 알지 못하지만 패킷의 주소가 kube-proxy에 의해 변경되어 원하는 Pod으로 전달된다.

#### kube-proxy
service port(inbound) => pod(outbound)
- 여타 proxy와 마찬가지로 user space에서 구동됨
- netfilter & iptables
	- proxy는 client, server 모두 연결을 맺을 때 interface device가 필요하다. 
	- kube-proxy에서 ethernet interfac 이거나 Pod내에 존재하는 가상 ethernet interface 두개 뿐이다. 
	- service는 pod, node처럼 대체될 수 있는 개체가 아닌 독립적이고 안정적이어야 한다. 또한 기존 네트워크 주소와 겹치지 않아야 한다. => 위의 2가지 interface device를 사용할 수 없다.
	- netfilter(rule-based 패킷 처리 엔진) + user space의 iptables을 통해 이를 해결

- netfilter
<img width="470" alt="image" src="https://user-images.githubusercontent.com/44525736/226187254-1829236a-0359-46bc-8949-86d83b496c79.png">
- netfilter를 옹해 service IP로 들어오는 packet을 kube-proxy 자신에게 라우팅 되도록 설정한다.

- iptables
<img width="477" alt="image" src="https://user-images.githubusercontent.com/44525736/226187272-91ab5a47-ac88-49f1-8daf-c2e598924179.png">

- user space로 들어오는 요청을 proxy하기 위해서는 다시 kernel space로 변환하는 비용이 발생한다.
- kubernetes 1.2 이후부터 iptables model을 통해 kube-proxy가 직접 proxy를 수행하지 않고 netfilter에게 이를 위임 (kube-proxy는 iptables를 수정하는 역할만 수행)
- 모든 패킷은 들어올 때 Hook(Netfilter)를 Trigger 하게 되고, 패킷들은 그 때 스택을 통해 통과된다. iptables는 이러한 netfilter hook들을 등록한다.

>현재 존재하는 kube-proxy mode
>- userspace mode: kube-proxy가 직접 routing 수행
>- iptables mode: kube-proxy가 iptables을 관리하는 역할만 수행
>- ipvs mode: userspace에서 구동되는 iptables이 아닌 kernel space에서 데이터 구조를 hash table로 관리


#### Routing != Loadblancing
위의 설명처럼 Service 트래픽은 netfilter를 통해 보내지기 때문에 L3에서 동작하게 된다. 따라서 외부에서 들어오는 클라이언트도 동일하게 IP+Port(L3)를 통해 연결을 시도해야 한다.

---- TODO: NodePort ,LoadBlanacer, Ingress
