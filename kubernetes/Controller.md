쿠버네티스 패턴 중 애플리케이션 관리 패턴을 대표하는 패턴을 **High-Level 패턴**이라고 말하며, **Controller 패턴**과 **Operator 패턴**이 이에 속한다.

### Controller 패턴

컨트롤러란 적어도 하나 이상의 쿠버네티스 오브젝트를 추적하며 오브젝트의 `status`를 `spec` 에 가깝게 만드는 역할을 한다. 일반적으로 컨트롤러는 API Server로 메세지를 보내 오브젝트를 관리하도록 지시하지만 직접 제어도 가능하다

- API Server를 통한 제어

    ex) Job 컨트롤러

- 직접 제어

    ex) 클러스터 오토 스케일링

    클러스터 외부의 것을 변경해야 하는 경우, 외부 시스템과 직접 통신해서 `status`를 `spec`에 가깝게 만든다.

컨트롤 루프들이 연결된 하나의 집합보다 간단한 컨트롤러를 여러개 사용하는 것이 쿠버네티스가 추구하는 방향성이다.

```
spec : 오브젝트의 특징(의도한 상태)
status: 오브젝트의 현재 상태
```

**참고 자료**

- [Kubernetes Document](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects)
- [Kubernetes Pattern](https://jflip.tistory.com/13)
