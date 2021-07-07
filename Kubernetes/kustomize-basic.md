## Kustomize 기본 정리
Kustomize는 kustomization 파일을 통해서 K8S 리소스를 커스터마이징 할 수 있는 도구
kubectl에도 내장되어 있긴 하지만 버전이 낮아서 독립적인 kustomize를 사용하는 것을 추천

### Directory Structure
```bash
├── base
│   ├── configMapGenerator.yaml
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── secretGenerator.yaml
│   └── service.yaml
└── overlays
    ├── prod
    │   └── kustomization.yaml
    └── dev
        └── kustomization.yaml
```

- base : `kustomization.yaml`과 함께 사용되는 directory로 사용자가 변경하기 원하는 리소스들을 포함
- overlay : `kustomization.yaml`만 존재하는 directory로 다른 base을 참조
base는 overlay에 대해 알지 못하고 여러 overlay에서 사용될 수 있다.

### SecretGenerator
secret resource를 직접 유지하는 것이 아니라 외부 파일 or env or literal을 통해 kustomize 실행 시점에 secret을 생성할 수 있다.


### ConfigMapGenerator
SecretGenerator처럼 외부 파일을 통해 ConfigMap을 만들 수 있도록 ConfigMapGenerator를 지원한다. 
만약 k8s resource로 configmap을 생성한다면 kustomize는 configmap이 변경되더라도 동일한 이름을 사용한다. 하지만 ConfigMapGenerator를 사용하면 kustomize가 hash를 이름에 자동으로 추가해주고 이를 통해서 rolling update를 트리거할 수 있다.

## 참고자료
- https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/kustomization/
- https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kustomize/
- https://github.com/kubernetes-sigs/kustomize/blob/master/examples/configGeneration.md
- https://github.com/kubernetes-sigs/kustomize
