BGP를 통해 Service에 외부 IP 설정이 가능하도록 할 수 있다.

### BGP
Border Gateway Protocol
서로 다른 AS(Autonomous System)를 연결해 주는 protocol
- AS: 하나의 네트워크 관리자에 의해 관리되는 라우터 그룹(도메인)
- eBGP: 두 AS를 연결하는 구간

#### Why BGP?
Cloud Vender(AWS, Azure, etc)에서 kubernetes를 사용하고 있다면 BGP를 사용하는 것이 의미 없을 수 있다. 하지만 Private Cloud 환경에서 kubernetes를 사용하고 있고 network를 유연하게 설정할 수 있으면 다음과 같은 시나리오에 대한 구현이 가능해질 수 있다.
- Routing 가능한 Pod
- Routing 가능한 Service IP

### Source IP
#### `NodePort`
 `externalTrafficPolicy: Local`로 해주어야 올바른 Client IP를 유지할 수 있다. (`externalTrafficPolicy: Cluster` 인 경우 node간 SNAT를 통해 source IP가 변경되기 때문에)
- https://kubernetes.io/docs/tutorials/services/source-ip/#source-ip-for-services-with-type-nodeport
> `externalTrafficPolicy: Local`
> 들어오는 요청을 같은 node에 있는 pod에만 요청을 routing 함

> `externalTrafficPolicy: Cluster`
> 클러스터 내의 아무 pod에 routing

#### Pros
cluster는 pod/service IP를 date center 내에서 routable하도록 advertise 할 수 있다.
- IP ACL에 pod의 IP 사용 가능
- packet encapsulation or custom rules on a Linux routing table을 제거할 수 있다.

## Reference
- https://www.asykim.com/blog/kubernetes-traffic-engineering-with-bgp
- https://kubernetes.io/docs/tutorials/services/source-ip/