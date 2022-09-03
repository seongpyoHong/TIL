### Reference
- https://www.sobyte.net/post/2022-04/controller-runtime/
- https://book-v1.book.kubebuilder.io/beyond_basics/controller_watches.html

### Architecture
#### Controller Runtime
- Cache 
	- resource를 watch하고 watch한 resource를 enqueue하기 위해 사용
	- API Server와 informer의 connection을 만드는 데 사용됨
- Controller
	- `eventHandler` 를 informer에 등록하고, queue에서 데이터를 받아옴
	- queue에서 가져온 데이터를 가지고 reconcile 로직 수행
- Informer: API Server와 통신하여 ListWatch 역할 수행 (필요한 리소스의 변경을 감지)
- Queue: controller가 처리할 데이터 저장

#### Made by User
- Manager
- Reconciler: user busniess logic


## Usage

### 1. Create Controller
```golang
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), crtl.Options{})
if err != nil {
	panic(err)
}

err = ctrl.NewControllerManagedBy(mgr).
      For(&corev1.Pod{}).
      Complete(&CustomReconciler{})

if err := mgr.Start(); err != nil {
...
}

type CustomReconciler struct {}

func (r customReconciler) Reconcile(ctx context.Context, request reconcile.Request) (request.Result, error) {
	// custom reconcil logic
	return reconcile.Result{}, nil 
}
```
- `NewControllerManagedBy` 를 통해 controller 생성
- `For` : watch할 resource
- `Complete` : reconclier 등록
- manager.Start를 통해 cache start

### 2. Set up EventHandler
informer는 등록된 eventHandler에 따라 어떤 이벤트를 감지하고, queue에 넣을지 결정한다.
```golang
mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), crtl.Options{})
if err != nil {
	panic(err)
}

c, _ := controller.New("custom", mgr, controller.Options{})
_ := c.Watch(&source.Kind{Type: &corev1.Pod{}}, &handler.EnqueueRequestForObject{}, predicate.Funcs{
  CreateFunc: func(event event.CreateEvent) bool {
	  ...
  },
})

```
queue에 넣을 이벤트를 정제하기 위해 custom eventHandler logic을 사용할 수 있다.

### 3. Set Cache selector
추가로 informer의 ListWatch 시 필요없는 resource를 줄이기 위해 label, field selector를 사용할 수도 있다.

---
### Cache를 통해 informer 생성
3가지 타입의 informer를 생성할 수 있다.
- `structured` : scheme과 일치하는 요청
- `unstructured` : scheme과 일치하지 않는 요청
- `metadata` : protobuf 형태의 요청

