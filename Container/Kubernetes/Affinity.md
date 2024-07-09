## Affinity
Kubernetes Resource 중 Pod을 특정 노드에서만 동작하도록 제한할 수 있다. 이를 수행하는 방법은 `nodeSelector` , `affinity` , `anfi-affinity` 방법이 존재한다.

### nodeSelector

node에 label을 추가한 후, `nodeSelector` 필드를 통해 특정 레이블을 가진 노드를 선택하여 Pod을 할당할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  **nodeSelector:
    disktype: ssd**
```

### affinity & anti-affinity

`nodeSelector` 보다 표현할 수 있는 종류가 확장된 설정으로 restrict 이외에도 soft / preference 규칙을 나타낼 수 있기 때문에 스케줄러가 규칙을 만족하지 못하더라고 계속해서 스케줄링 할 수 있다.

또한, `nodeSelector`은 node에 대한 label 값을 사용하기 떄문에 node에 대한 제한만 가능했지만, `affinity & anti-affinity`는 다른 Pod에 대응하는 규칙을 정의할 수 있다.

- **nodeAffinity**

    nodeSelector와 유사한 기능 (node 제한)

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: with-node-affinity
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/e2e-az-name
                operator: In
                values:
                - e2e-az1
                - e2e-az2
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: another-node-label-key
                operator: In
                values:
                - another-node-label-value
      containers:
      - name: with-node-affinity
        image: k8s.gcr.io/pause:2.0
    ```

    requiredDuringSchedulingIgnoredDuringExecution : restrict한 조건을 의미하며 만약 조건을 만족하지 않는다면 스케줄링을 중단한다.

    preferredDuringSchedulingIgnoredDuringExecution : preference한 조건을 의미하며 조건을 만족하지 않아도 스케줄링이 계속 진행되지만, 조건의 만족을 언제나 보장하지 않는다.

- **podAffinity**

    실행 중인 Pods의 레이블을 기반으로 Pod이 스케줄링 될 수 있는 노드를 제한한다.

    ```yaml
    spec:
        affinity:
          podAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  app.kubernetes.io/app: graylog 
    ```

여기서 `topologyKey` 는 Pod을 배포하는 범위를 나타낸다. hostname 인 경우 같은 hostname을 가지는 node에 배포해야하며, `region`, `rack` 과 같이 배포 범위를 넓힐 수 있다.

- **PodAntiAffinity**

    PodAffinity와 동일하게 실행중인 Pods의 레이블을 기반으로 Pod이 스케줄링 될 수 있는 노드를 제한다. PodAntiAffinity는  해당 조건을 만족하는 Pod이 존재하지 않는 topology로 pod을 배포한다.

    ```yaml
    spec:
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  app.kubernetes.io/app: graylog 
    ```

### 참고자료

- [https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/)
- Kubernetes in Action - 16장
