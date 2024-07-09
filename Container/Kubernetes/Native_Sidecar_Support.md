 ### 기존 Sidecar의 문제 
- Application이 Sidecar보다 먼저 뜨면 Sidecar에서 수행해야하는 작업(e.g. network 접속 in istio sidecar)을 수행하지 못한다. (race condition)
	- postStart hook 같이 lifecycle hook으로 조정은 가능하긴 하지만..
- Application이 즉어도 Sidecar는 계속해서 떠있다.
- InitContainer는 Sidecar의 기능을 이용할 수 없음
- Sidecar Update가 전체 Pod의 재시작으로 야기함
=> Container Lifecycle 관련 문제가 존재

### Before 1.28
- Pod Lifetime > Sidecar Lifetime: initContainer 사용
	- sidecar가 다른 initContainer나 main container가 시작되기 전에 종료되어야만 함
- Pod Lifetime < Sidecar Lifetime: main container에서 application container 함께 동작하도록 사용
	- main container 간의 startup order를 설정할 수 없다.
	- application container가 종료된 후에 sidecar container로 인해 Pod 삭제가 block될 수 있다.

### Native Support in K8S 1.28
`SidecarContainers` feature gate를 enable 하고, init container의 restartPolicy를 `Always`로 설정하면 된다.
```yaml
apiVersion: v1
kind: Pod
spec:
  initContainers:
  - name: secret-fetch
    image: secret-fetch:1.0
  - name: network-proxy
    image: network-proxy:1.0
    restartPolicy: Always # This is Sidecar
```

#### 어떻게 동작?
main container가 시작되기 전에 sidecar container가 시작되고, main container가 제거된 이후 sidecar container가 종료된다.

#### 그림 설명 (https://buoyant.io/blog/kubernetes-1-28-revenge-of-the-sidecars)
- Before 1.28 (without Sidecar Container)
<img width="833" alt="image" src="https://github.com/seongpyoHong/TIL/assets/44525736/8ebf48db-0233-4342-b062-b21189075698">

- After 1.28 (with Sidecar Container)
<img width="855" alt="image" src="https://github.com/seongpyoHong/TIL/assets/44525736/12d70fba-409b-4a5c-8e28-67c94535838f">

### 아직 남아있는 문제
- Sidecar Container도 Pod에서 같이 보이는 점 (sidecar pattern 자체의 문제)
- alpha version이기 때문에 리소스 사용량에 대한 계산이 빠져있다고 함 (pod의 resource request가 실제보다 낮은 것처럼 동작)

### Reference
- https://kubernetes.io/blog/2023/08/25/native-sidecar-containers/
- https://istio.io/latest/blog/2023/native-sidecars/
