쿠버네티스 패턴 중 애플리케이션 관리 패턴을 대표하는 패턴을 고급 패턴이라고 말하며, Controller 패턴과 Operator 패턴이 이에 속한다.

### Operator란?

사용자 정의 리소스를 사용하여 어플리케이션 및 해당 컴포넌트를 관리하는 kubernetes extension이다.

**목적**

CR를 사용할 때, 쿠버네티스에서 제공하는 것 이상의 작업을 자동화하기 위한 목적으로 Operator를 사용한다. Operator는 Controller 패턴을 통해 쿠버네티스 내부 코드를 수정하지 않아도 클러스터의 동작을 확장할 수 있다. 


**참고사항** 
- **Controller vs Operator**
  - Operator는 CRD(사용자 정의 리소스)로 구현된 Customized Controller(https://stackoverflow.com/questions/47848258/kubernetes-controller-vs-kubernetes-operator)

- CRD ⇒ CR을 생성하기 위한 정의, Controller ⇒ CR의 변경 감시 및 원하는 상태로 만들기위한 요청 전송


**Custom Operator 생성**

Operator를 개발하기 위한 도구로 KUDO, kubebuilder, Operator Framework 등이 존재한다. 이번에는 Operator Framework를 통해 Operator를 생성해보자. [Operator SDK 문서](https://sdk.operatorframework.io/docs/overview/_overview/)를 살펴보면 Golang, Ansible, Helm을 사용한 operator developer workflow를 제공한다. 



**Golang Operator**

Golang Operator는 쿠버네티스의 Custom Resource를 관리하는 Controller의 집합을 의미한다.  따라서, Golang Operator를 개발하는 과정은 1. Custom Resource를 생성, 2. Controller 개발 단계로 이루어진다.

Operator SDK를 사용하게 되면 CR이나 Controller에서 발생하는 Boiler Plate를 자동으로 생성해주기 떄문에 개발자는 핵심 로직 개발에 집중할 수 있다.

[https://ssup2.github.io/programming/Kubernetes_Operator_SDK_Golang/](https://ssup2.github.io/programming/Kubernetes_Operator_SDK_Golang/)을 참고하면 Operator SDK를 통해 개발한 Controller는 다음과 같은 주요 패키지를 가진다.

- Runtime Controller Package

    Controller가 관리할 Custom Resource의 변경을 감지하여, SDK Controller Package의 Reconcile Loop가 동작하도록 한다.

- Runtime Manager Package

    Kubernetes Client 및 Cache를 초기화한다.

- SDK Controller Package

    실제 Controller 로직을 수행하는 Reconcile Loop와 Runtime Manager Package로 부터 전달받은 Kubernete Client를 가지고 있다. 
    
    
    
### Initialize
Operator SDK Tutorial에 있는 Memcached Operator를 만들어보자. 먼저 working directory를 생성한 후, 초기화 작업을 수행한다. operator-sdk는 

```bash
▶ operator-sdk init --repo=github.com/seongpyoHong/memcached-operator
▶ ls
Dockerfile Makefile   PROJECT    bin        config     go.mod     go.sum     hack       main.go
```

operator를 위한 main program은 `main.go`로 Manager의 초기화 및 실행을 담당한다.

**main.go**

```go
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
		Scheme:             scheme,
		MetricsBindAddress: metricsAddr,
		Port:               9443,
		LeaderElection:     enableLeaderElection,
		LeaderElectionID:   "9dd21db1.my.domain",
		//Namespace: namespace
	})
```

추가로 Controller가 관찰할 리소스를 제한하기 위해 `Options`에 Namespace 관련 설정을 추가할 수 있다.

다음으로 Memcached-Operator를 통해 관리할 Memcached CRD API와 Controller를 생성한다. 

```bash
▶ operator-sdk create api --group=cache --version=v1alpha1 --kind=Memcached
Create Resource [y/n]
y
Create Controller [y/n]
y
...
...
```

### **Memcached CRD API**

`api/v1alpha1` 아래에 `memcached-types.go` 에 생성된다. 이 파일에 CRD가 가져야 하는 정보를 추가해준다. Spec의 Size는 동작해야 하는 Memcached Pod의 개수를 나타내고. Status의 Nodes는 Memcached가 동작하는 Pod의 이름을 나타낸다.

```go
package v1alpha1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
// NOTE: json tags are required.  Any new fields you add must have json tags for the fields to be serialized.

// MemcachedSpec defines the desired state of Memcached
type MemcachedSpec struct {
	Size int32 `json:"size"`
}

// MemcachedStatus defines the observed state of Memcached
type MemcachedStatus struct {
	Nodes []string `json:"nodes"`
}

// +kubebuilder:object:root=true
// +kubebuilder:subresource:status

// Memcached is the Schema for the memcacheds API
type Memcached struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   MemcachedSpec   `json:"spec,omitempty"`
	Status MemcachedStatus `json:"status,omitempty"`
}

// +kubebuilder:object:root=true

// MemcachedList contains a list of Memcached
type MemcachedList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata,omitempty"`
	Items           []Memcached `json:"items"`
}

func init() {
	SchemeBuilder.Register(&Memcached{}, &MemcachedList{})
}
```

API 생성 후, 업데이트를 위해 다음 커맨드를 실행한다.

```bash
> make generate
```

마지막으로  CRD의 manifest를 생성한다.

```bash
> make manifest
```

위의 명령어를 실행하면 `config/crd` 아래에 CRD가 생성된다.

### **Memcacehd Controller**

`controllers/memcached-controller.go` 에 controller boilerplate가 생성된다. Memcached CR을 위해 Controller가 실행해야할 핵심 로직은 다음과 같다.

- Memcached Deployment 생성
- Deployment replica가 spec의 size와 동일하도록 보장
- Memcached CR의 status를 업데이트

이제 controller.go에 생성된 함수를 하나씩 살펴보자. 먼저 `SetupWithManager` 함수는 컨트롤러가 소유하고 관리하는 CR 및 기타 리소스를 감시하기 위해 어떻게 컨트롤러가 구축되는지를 지정한다.

```go
func (r *MemcachedReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&cachev1alpha1.Memcached{}).
		Owns(&appsv1.Deployment{}).
		WithOptions(controller.Options{
			MaxConcurrentReconciles: 2,
		}).Complete(r)
}
```

`NewControllerManagedBy()` 는 다양한 컨트롤러 설정을 할 수 있는 controller builder를 제공한다.

`For(&cachev1alpha1.Memcached{})` 컨트롤러가 관찰하기 위한 우선적인 리소스를 Memcached 타입으로 특정한다. Memcached 타입의 Add/Update/Delete 이벤트에 대해 Reconcil 루프가 해당 Memcached 개체에 대한 Reconcil 요청을 전송한다.

`Owns(&appsv1.Deployment{})` 컨트롤러가 관찰하기 위한 추가 리소스로 Deployments를 지정한다. 이 경우에는 Memcached Object가 생성한 Deployment를 관찰하는 것을 의미한다. 

`WithOptions` 를 통해 Controller에 적용할 다양한 옵션을 추가할 수 있다. 위의 코드는 Reconcil을 동시에 최대 2개 수행한다는 것을 의미한다.

다음으로 `Reconcile(req ctrl.Request)` 를 살펴보자. 이 함수는 Request를 받았을 때, 어떤 동작을 수행할지 정의하는 부분이다. 자세한 코드는 [https://github.com/operator-framework/operator-sdk/blob/master/example/memcached-operator/memcached_controller.go.tmpl](https://github.com/operator-framework/operator-sdk/blob/master/example/memcached-operator/memcached_controller.go.tmpl) 에 존재하며, 간단하게 설명하면 위에서 설명한 Controller가 실행할 핵심로직을 수행한다.

### Build

이제 operator를 빌드한 후 실행시켜보자.

```bash
> make install
go: creating new go.mod: module tmp
go: found sigs.k8s.io/controller-tools/cmd/controller-gen in sigs.k8s.io/controller-tools v0.3.0
/Users/seongpyo/workspace/go/bin/controller-gen "crd:trivialVersions=true" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
go: creating new go.mod: module tmp
/Users/seongpyo/workspace/go/bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/memcacheds.cache.my.domain created

> kubectl get crd
NAME                                        CREATED AT
managedcertificates.networking.gke.io       2020-10-02T05:32:57Z
```

CRD가 생성된 것을 확인할 수 있다.

### Run

Operator를 실행하는 방법은 2가지가 존재한다.

- Local에서 Go 프로그램으로 실행
- Cluster에 Deployment로 배포

이번에는 Local에서 실행시키는 방법을 사용해보자.

```bash
▶ make run ENABLE_WEBHOOKS=false
go: creating new go.mod: module tmp
go: found sigs.k8s.io/controller-tools/cmd/controller-gen in sigs.k8s.io/controller-tools v0.3.0
/Users/seongpyo/workspace/go/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
go fmt ./...
go vet ./...
/Users/seongpyo/workspace/go/bin/controller-gen "crd:trivialVersions=true" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
go run ./main.go
2020-10-02T15:07:51.303+0900	INFO	controller-runtime.metrics	metrics server is starting to listen	{"addr": ":8080"}
2020-10-02T15:07:51.304+0900	INFO	setup	starting manager
2020-10-02T15:07:51.304+0900	INFO	controller-runtime.manager	starting metrics server	{"path": "/metrics"}
2020-10-02T15:07:51.304+0900	INFO	controller	Starting EventSource	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "source": "kind source: /, Kind="}
2020-10-02T15:07:51.406+0900	INFO	controller	Starting EventSource	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "source": "kind source: /, Kind="}
2020-10-02T15:07:51.508+0900	INFO	controller	Starting Controller	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached"}
2020-10-02T15:07:51.508+0900	INFO	controller	Starting workers	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "worker count": 1}
```

다음으로 Operator가 관찰할 CR을 생성한다.

```bash
▶ kubectl apply -f cache_v1alpha1_memcached.yaml
memcached.cache.my.domain/memcached-sample created
```

CR을 생성하면 Operator를 실행했던 쉘에서 이를 감지한다.

```bash
2020-10-02T15:09:07.588+0900	INFO	controllers.Memcached	Creating a new Deployment	{"memcached": "default/memcached-sample", "Deployment.Namespace": "default", "Deployment.Name": "memcached-sample"}
2020-10-02T15:09:08.243+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:08.256+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:08.256+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:14.221+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:14.254+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:14.263+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:14.263+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
2020-10-02T15:09:15.818+0900	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "cache.my.domain", "reconcilerKind": "Memcached", "controller": "memcached", "name": "memcached-sample", "namespace": "default"}
```

### 참고 자료
- [https://m.blog.naver.com/alice_k106/221586279079](https://m.blog.naver.com/alice_k106/221586279079)
- [https://ssup2.github.io/programming/Kubernetes_Operator_SDK_Golang/](https://ssup2.github.io/programming/Kubernetes_Operator_SDK_Golang/)
