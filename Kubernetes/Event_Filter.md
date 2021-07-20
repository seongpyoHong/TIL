### Event Filter란?

Controller를 생성할 때 특정 이벤트에 대한 Predicate 함수를 수행할하기 위한 함수

Builder 패키지를 보면 `withEventFilter`라는 함수가 존재한다. 이 함수는 특정 이벤트를 필터링하는 `Predicate` 함수를 지정할 수 있고, `Predicate` 인터페이스는 다음과 같은 함수를 갖는다.

```go
type Predicate interface {
    // Create returns true if the Create event should be processed
    Create(event.CreateEvent) bool

    // Delete returns true if the Delete event should be processed
    Delete(event.DeleteEvent) bool

    // Update returns true if the Update event should be processed
    Update(event.UpdateEvent) bool

    // Generic returns true if the Generic event should be processed
    Generic(event.GenericEvent) bool
}
```

- Operator SDK의 예제

```go
func ignoreDeletionPredicate() predicate.Predicate {
	return predicate.Funcs{
		UpdateFunc: func(e event.UpdateEvent) bool {
			// Ignore updates to CR status in which case metadata.Generation does not change
			return e.ObjectOld.GetGeneration() != e.ObjectNew.GetGeneration()
		},
		DeleteFunc: func(e event.DeleteEvent) bool {
			// Evaluates to false if the object has been confirmed deleted.
			return !e.DeleteStateUnknown
		},
	}
}

func (r *MemcachedReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&cachev1alpha1.Memcached{}).
		Owns(&corev1.Pod{}).
		WithEventFilter(ignoreDeletionPredicate()).
		Complete(r)
}
```

위의 예제는 다음과 같은 이벤트에 대해서 필터링을 진행한다.

- Delete 이벤트가 발생했을 때, deleteState가 Unknown인 이벤트(삭제가 확인되지 않은 리소스)
- Update 이벤트가 발생했을 때, 이전 metadata의 Generation이 갱신되지 않은 이벤트(업데이트가 수행되지 않은 리소스)

문서에 따르면 이를 통해서 Reconcile 함수가 API 서버로 보내는 요청의 수를 줄일 수 있다고도 한다.

### Reference

[Kubebuilder Event Filter](https://stuartleeks.com/posts/kubebuilder-event-filters-part-1-delete/)

[Operator SDK Evnet Filter](https://sdk.operatorframework.io/docs/building-operators/golang/references/event-filtering/)

[sigs.k8s.io/controller-runtime/pkg/builder#Predicates (v0.9.2)](https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.9.2/pkg/builder#Predicates)
