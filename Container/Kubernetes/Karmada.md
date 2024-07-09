### Limitation with KubeFed
- KubeFed v1
	multiple cluster로 service를 분산시킬 수는 있지만 다른 종류의 object들을 handling 하지 못함 => Archive

- KubeFed v2
	하나의 control plane (main cluster)가 존재하고 다른 cluster의 resource를 관리
	- 한계
		- 별도의 Federated API에 대한 학습이 필요
		- 확장성이 부족

### Karmada (Kubernetes Armada)
Application의 변경 없이 multiple Kubernetes cluster로 관리 가능
- Resource Template
- Propagation Policy : https://github.com/karmada-io/karmada/blob/master/samples/nginx/propagationpolicy.yaml
- Override Policy : https://github.com/karmada-io/karmada/blob/master/samples/nginx/overridepolicy.yaml

Karmada API Server가 있고, 각 Controller 존재

- 동시에 수정됐을 때 conflict는?