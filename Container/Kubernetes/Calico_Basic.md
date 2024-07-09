CNI 중 하나인 Calico에 대해 알아보기

### Calico Architecture
#### Calico API server
Calico resource를 `kubectl`를 통해 직접 관리할 수 있도록 해주는 API Server

#### Felix
route와 ACL 및 host에서 원하는 연결을 위해 요구되는 작업들을 위한 Program
- Interface Management
- Route Programming
- ACL Programming
- State reporting

#### BIRD
Felix로부터 route를 가져와서 BGP peer에게 분배한다. (Vxlan mode에서는 존재 X)

#### CONFD
config management tool

#### CNI Plugin
Provides Calico networking for Kubernetes clusters.
- Calico Network를 CNI spec에 맞춰 사용할 수 있게 해주는 binary
- 모든 Node에 설치되어야 한다.

#### Datastore Plugin
Datastore에 대한 각 노드의 영향을 줄임으로써 확장성을 높이기 위한 component
- Kubernetes API datastore (kubernetes)
- etcd (non kubernetes)

#### Typha
datastore과 Felix Instance 사이에서 동작 (default: disable)
- datastore cache
- felix resource usage 감소를 위한 plugin (kubernetes node가 많아지는 경우 사용)

### Deployment
- daemonset으로 calico-node가 동작, 해당 pod에 bird, felix, confd가 동작
- deployment로 calico-controller 배포

BGP를 통해 각 노드에 할당된 Pod 대역의 정보를 Bird를 통해 전달
=> Felix가 routing table 및 iptable rule에 전달받은 정보를 주입
=> confd는 변경된 값을 감지하여 반영할 수 있도록 하는 역할

### Calicoe Network Mode
- IPIP (Default)
- Vxlan
	- BGP를 사용하지 않기 때문에 잦은 BGP 업데이트로 인한 부하를 줄일 수 있다. (BIRD component X)
	- UDP 기반

### Reference
- https://projectcalico.docs.tigera.io/about/about-calico
- https://www.notion.so/CloudNet-Blog-c9dfa44a27ff431dafdd2edacc8a1863