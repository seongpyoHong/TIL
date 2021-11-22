### 기존에 존재하던 affinity의 한계

- PodAffinity
    
    조건을 만족하는 topology domain에 pod이 무한정 스케줄링이 됨
    
- PodAntiAffinity
    
    하나의 topology domain에 pod이 하나만 스케줄링 됨
    

### Design

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  topologySpreadConstraints:
    - maxSkew: <integer>
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
```

- maxSkew : 균등하게 분산되지 않을 수 있는 최대 차이
- topologyKey : node label
- whenUnsatisfiable
    - DoNotSchedule
    - ScheduleAnyway
- labelSelector : Pod selector

### with NodeSelctor or NodeAffinity

skew 계산에서 해당 노드를 생략

**Deployment scale down과 같이 Pod의 분포가 불균형해지면?**
[descheduler](https://github.com/kubernetes-sigs/descheduler) 를 통해 재분배가 가능하다.

### Reference
- https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/
- https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/895-pod-topology-spread#motivation
