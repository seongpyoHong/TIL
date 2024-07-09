### Background
사내 CSI Driver를 업그레이드하면서 내부 동작 방식에 대해 모르고 있던 부분이 있어 정리

### Objective
Storage Provider (SP)가 Container Orchestration의 종류에 상관없이 동일하게 사용할 수 있는 Storage API를 만들게 하기 위함 (Interface로 정의)
- Dynamic Provisioning/De-Provisioning
- Attach / Detach
- Mount / Unmount
- Comsumption of block & moutable volume
- Local Storage Provider
- Create / Delete snapshot


#### Architecture
CO는 Controller Plugin(centralized)과 Node(headless) Plugin을 모두 handling 할 수 있어야 한다. 

- Controller Plugin (StatefulSet)
	Volume을 Create/Delete & Attach/Detach 하는 plugin
	- 1개 이상의 controller가 존재하는 경우, 동일한 볼륨을 중복해서 생성하려고 할 수 있기 때문에 single copy로 존재해야 한다.
- Node Plugin (DaemonSet)
	Volume이 attach 되기 위해서 format, mount 과정이 필요하기 때문에 volume이 provisioning되는 모든 node에 배포될 필요가 있다.

Controller Plugin과 Node Plugin이 배포되는 방식은 다음과 같은 방법들이 존재한다.
- controller: master node / node: worker node
- controller & node: worker node / seperated by service
- controller & node : worker node / unifed service
- only node plugin in worker node
=> identity RPC는 각 plugin에 모두 필요 


### RPC spec
CSI RPC는 3가지 Service로 구성된다.
- Identity Service RPC
	CO가 plugin의 capability, health, metadata에 대한 query를 가능하게 한다.

- Controller Service RPC (Controller Plugin)
	- volume 관리에 대한 책임 (create/delete, attach/detach, snapshot)
- Node Service RPC (Node Plugin)
	- Volume이 provisioning 될 node에서 동작

#### Question 1. Stage Volume vs Publish Volume
https://github.com/kubernetes-csi/docs/issues/24
Volume을 global directory에 먼저 mount하기 위한 방법 => 이후 해당 path를 pod directory에 mount 해야 한다. (NodePublishVolume을 통해)
- 필요한 이유: NFS나 모든 pod이 동일한 노드에서 사용되는 경우, 여러 pod이 동일한 volume을 사용할 수 있기 때문에 2 phase mount를 통해 이를 가능하게 한다. (또한 위의 경우 disk format할 필요도 있기 때문에)


### Sidecar
CO가 어떤 Pod이 Node이고 Controller인지 알 수 있는 방법은?
=> 이를 해결하기 위한 sidecar container들이 존재
- driver-registrar(node plugin): CSI Driver를 kubelet에 등록, driver custom node을 label로 등록
- external-provisioner(controller plugin): PVC를 watch하며, CSI endpoint에 대해 Create/Delete 요청을 trigger
- external-attacher(controller plugin): VolumeAttach를 watch하며, CSI endpoint에 대해 ControllerPublish/Unpublish 요청을 trigger

### Reference
- spec: https://github.com/container-storage-interface/spec/blob/master/spec.md
- https://arslan.io/2018/06/21/how-to-write-a-container-storage-interface-csi-plugin/