### Reference
https://www.youtube.com/watch?v=jHHcAEOaja4&ab_channel=CNCF%5BCloudNativeComputingFoundation%5D

### wasmCloud?
wasm orchestrator with declarative
- wasm => run application anywhere
- wasmCloud => lets your wasm components talk to each other the same way

어떻게 오케스트레이션을 진행할 수 있는가?
- k8s, nomad? 
	- wasmCloud를 돌리기에 한계점 존재
- wadm(wasmCloud application deployment manager)
  - https://github.com/wasmCloud/wadm
  - controller 역할이라고 이해 (reconcile loop)
 
### Architecture
<img width="512" alt="image" src="https://github.com/seongpyoHong/TIL/assets/44525736/69a0d78f-ac5a-4ca2-aafb-3288e61fc6fc">
- NATS Jetstream 사용

<img width="517" alt="image" src="https://github.com/seongpyoHong/TIL/assets/44525736/3ac38b27-5e61-4d73-9b83-073d0be53110">
- OAM Model의 YAML이 input
- 이를 Parsing 하여 Scaler라는 단위로 나눔
- 각 Scaler에 대해 Cloud Event를 발행 (to NATS Jetstream)
