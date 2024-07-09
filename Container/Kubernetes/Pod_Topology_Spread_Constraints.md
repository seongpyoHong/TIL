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

---
## MatchLabelKeys

### Background

Pod을 AZ 별로 균등하게 나누어 배포하기 위해 다음과 같이 topologySpreadConstrains를 설정
```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: kubernetes.io/hostname
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      availability-zone: az-1
```

하지만, Deployment Rolling Update 도중에 이전 Revision의 Pod과 현재 Revision의 Pod이 모두 labelSelector의 대상이 되어 AZ에 균등하게 분배되지 않는 상황이 발생

### Solution
찾아보니 관련 [이슈](https://github.com/kubernetes/kubernetes/issues/105661)가 존재했고, 이를 통해 파생된 KEP를 확인해본 결과 `matchLabelKeys` 라는 기능을 통해 해결된 것을 확인하였다.

문서에도 확인할 수 있듯이 matchLabelKeys은 spread를 계산하기 위한 label key 목록이다. 이를 통해 Deployment Controller에서 Pod에 찍어주는 `pod-template-hash` 를 통해 revision을 구분할 수 있도록 설정할 수 있다.

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: kubernetes.io/hostname
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      availability-zone: az-1
  matchLabelKeys:
    - pod-template-hash
```
