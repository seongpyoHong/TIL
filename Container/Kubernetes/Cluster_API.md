## What is Cluster API?
extensible API to create Kubernetes cluters across multiple infrastructure

- Management cluster : Cluster API 컴포넌트가 설치된 클러스터
- Workload cluster: Management cluster를 통해 생성 및 관리되는 클러스터
- Provider: underlying IaaS platform (e.g. AWS, Azure, GCP)

## How does Cluster API work?
<img width="693" alt="image" src="https://user-images.githubusercontent.com/44525736/223114111-7cdef1b3-d0a7-4d10-a2a5-f89580ffaa7f.png">

Management cluster에는 다음과 같은 controller manager가 존재
- Core Controller Mananger
- Bootstrap Controller Manager
- Infrastructure Controller Manager
- Control Plane Controller Manager

### CRDs and Spec
<img width="680" alt="image" src="https://user-images.githubusercontent.com/44525736/223114244-3fbaa817-cf36-4749-ad53-dba89d69f9f5.png">

<img width="683" alt="image" src="https://user-images.githubusercontent.com/44525736/223114201-c8e0e8d3-35d9-47d2-ba03-22c26181bc74.png">


> bootstrap controller
> - bootstrap & init & join
> 
#### Master node
1. `Cluster` CR을 생성하면 Cluster Controller가 `Cluster` spec에 정의된 `controlPlaneRef`, `InfrastructureRef` 에 해당 하는 CR의 OwnerReference를 설정
2. `Infrastructure controller` 는 `InfrastructureCluster` CR의 spec에 정의된 infra-specific resource들을 생성한다. (e.g. Control Plane LB, VPC, AZ)
3. 생성되면 Status를 변경 => Core Controller가 `Cluster`의 status(`infrastructureReady: true`)를 변경
4. `ControlPlane controller`는 `Cluster` 의 `infrastructureReady: true`인지 확인한 후, `Machine` 을 생성할 수 있는지 확인
	 - `kubeconfig`를 secret으로 제공해줌
	 - control plane 생성이 완료되면 `Cluster`의 status를 변경
	- `Machine` CR의 spec이 변경되면 `Core Controller` 는 연관된 `InfrastructureMachine` , `KubeadmConfig`  의 ownerReference 설정 => InfraStructureMachine가 준비되면 Bootstrap controller가 master node 구성

#### Worker node
1. Machine & InfrastructureMachine 생성 => Bootstrap controller가 worker node 생성

## Reference
- https://www.youtube.com/watch?v=7wdVPuf-gXI
- https://speakerdeck.com/kakao/kubeonetiseu-peonhago-kulhago-segsihage-mandeulgi
- https://if.kakao.com/2022/session/58
