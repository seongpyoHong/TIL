## Cloud Provider
- infrastructure provider와 integration을 제공

#### In-Tree vs Out-of-Tree
- In Tree
	 - k8s components에 cloud provider에 대한 context가 담겨있음
- Out of Tree (de-facto)
	- 별도의 cloud-controller-manager가 필요한 구조 (Cloud specific한 요청을 담당함)

기존의 In Tree 방식은 확장성이 부족
- k8s 발전 방향은 ubiquitous임
- 대부분의 infrastructure provider는 kubernetes를 native하게 지원하기를 원하면서 In Tree 방식으로 진행하게 되면 각 provider에 대한 의존성이 증가함


---

### Openstack manila CSI plugin
https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/manila-csi-plugin/using-manila-csi-plugin.md

Manila csi plugin은 2개의 main component로 구성된다.

#### Controller
StatefulSet으로 배포되고 CSI Manilia와 아래의 컴포넌트를 실행시킴
- external-provisiner: https://github.com/kubernetes-csi/external-provisioner
	- `CreateVolume`, `DeleteVolume` function을 호출하여 dynamic provision을 수행하는 sidecar

- external-snapshotter: https://github.com/kubernetes-csi/external-snapshotter
	- Sidecar container that watches Kubernetes Snapshot CRD objects and triggers CreateSnapshot/DeleteSnapshot against a CSI endpoint.


#### Node plugin
DaemonSet으로 배포되고 CSI Manila와 아래의 컴포넌트를 실행시킴
- node-driver-register: https://github.com/kubernetes-csi/node-driver-registrar
	-  registers the CSI driver with Kubelet using the [kubelet plugin registration mechanism](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/#device-plugin-registration).


## Reference 
- https://kubernetes.io/blog/2019/04/17/the-future-of-cloud-providers-in-kubernetes/
- https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/openstack-cloud-controller-manager/using-openstack-cloud-controller-manager.md
