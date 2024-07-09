## Reference
https://www.cncf.io/blog/2021/12/20/introduction-to-multi-tenancy-in-kubernetes/

## Multi Tenency?
여러개의 tenant를 하나의 instance에서 공유하여 사용하는 방법
computing resource 및 cost를 줄이기 위해서 필요

> 물론 돈이 많으면 클러스터를 하나씩 주면 되지만, 그렇지 않기에..
> 그리고 클러스터를 제공하는 방식은 관리 비용 또한 증가하게 된다. (보안 등)

## Architecture
여기서 소개하는 architecture는 다음과 같다.
![image](https://user-images.githubusercontent.com/44525736/146939073-2ef8cd00-2979-430f-998b-33233382b0f0.png)

- A : Cluster as a Service
- B : Namespace as a Service
  - cluster-scope resource 사용이 유저에게 제한
  - control plane을 공유해서 사용하게 됨 (다른 tenant에 영향을 받는다.)
- C : 분리된 control plane을 제공하는 방법 ([Virtual Cluster](https://github.com/kubernetes-sigs/cluster-api-provider-nested/tree/main/virtualcluster), [vCluster](https://github.com/loft-sh/vcluster))
- D : Tenant라는 상위 개념을 사용하는 방법?

## vCluster?
![image](https://user-images.githubusercontent.com/44525736/146941414-9bde02c0-07f0-48ff-8281-d2f0dd8fff17.png)

- Control Plane
  API Server, Controller Manager, data-store connection 담당
  > default data store가 sqlite라고 함 (etcd, mysql 같은 다른 소스도 사용 가능[link](https://www.vcluster.com/docs/operator/external-datastore))
- Syncer : virtual cluster에 있는 pod을 복사해서 host cluster에 scheduling 되도록 함 

### Resource Type
vCluster에서는 2가지로 리소스를 구분함
- High Level (e.g. Deployment, Statefulset, CRD 등)
  vCluster CP에 의해서 관리 => 실제 생성되는 low-level 리소스는 host cluster의 host namespace에 배포
- Low Level (e.g. Pod, ConfigMap)
  Syncer에 의해서 유지 => 이것도 실제 생성은 host cluster의 host namespace에 배포
  
 
### Advantage?
1. High level resource는 vCluster에만 유지되므로 host cluster의 request가 적어진다
2. cluster-wide permission이 유저에게 필요 없어짐
3. 하나의 host namespace으로 캡슐화 가능 => cleanup도 쉬움

### TODO
PoC 해보기
