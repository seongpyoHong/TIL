### Background
A Cluster에서 Remote Cluster의 K8S resource 이벤트를 받아서 controller의 reconcile을 trigger 해야함.

### Solution
controller-runtime에서는 다른 클러스터로 부터 cache(controller-runtime에서 informer를 추상화한 것)를 받을 수 있는 구현체를 제공한다.
```go
Watches(
	source.NewKindWithCache(&corev1.Secret{}, mirrorCluster.GetCache()),
	&handler.EnqueueRequestForObject{},
).
```

다만 한 가지 확인해야할 부분은 `reconcile.Request` 는 다음과 같은 구조로 Event가 발생한 Namespace와 Name만 받을 수 있다. 
```go
// Request contains the information necessary to reconcile a Kubernetes object. This includes the// information to uniquely identify the object - its Name and Namespace. It does NOT contain information about  
// any specific Event or the object contents itself.  
type Request struct {  
	// NamespacedName is the name and namespace of the object to 
	reconcile.types.NamespacedName  
}

type NamespacedName struct {  
	Namespace string  
	Name string  
}
```

따라서 eventhandler에서는 remote cluster에서 변경이 발생한 resource의 정보만 알 수 있다. 만약 remote cluster에서 변경이 발생한 리소스에 대해 다른 리소스의 reconcile을 트리거 하고 싶다면 label이나 annotation을 해당 정보를 저장해서 가져오는 방식 등과 같이 별도의 eventhandler를 `EnqueueRequestsFromMapFunc` 통해 구현해야 한다. 

TMI) cluster 정보를 알기 위해서는 다음과 같은 방법도 사용할 수 있다. (https://www.sobyte.net/post/2022-11/k8s-multi-cluster-operator/)

### Reference
- https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/source#NewKindWithCache

