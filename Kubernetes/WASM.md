### Assembly Language
- 컴파일러가 어떤 언어든지 단일 IR(Intermediate Representation)로 번역하여 대상 아키텍쳐의 어셈블리 코드로 번역

### WebAssembly
프로그래밍 언어를 특정 기계(e.g. arm, amd)에 종속되지 않고 기계에 맞는 기계어로 변경해줄 수 있게 해줌
- browser가 WASM module을 다운로드하면 WASM으로 부터 대상 기계의 어셈블리 코드를 위한 hop을 만들어낼 수 있다.
- Interface의 등장(WASI)으로 인해 `.wasm` module을 Browser 뿐만 아니라 WASI를 만족하는 WASM runtime 어느 곳에서든지 실행할 수 있음 => secure, portable, and efficient virtual machine (VM) sandbox
	- WASM Runtime: https://wasmedge.org/, https://wasmer.io/

### **Container 대신 Wasm?**
Container의 하나의 타입으로 WASM module을 사용하는 방향으로 발전하고 있는거 같음
- CRI 중 하나인 Containerd에서 wasm workload를 실행할 수 있도록 하는 설정:  https://github.com/second-state/wasmedge-containers-examples/blob/main/containerd/README.md
- WASM workload를 돌릴 수 있는 node로 scheduling 해주는 WASM용 kubelet: 
	- https://krustlet.dev/
	- https://github.com/containerd/runwasi

### **Pros?**
- container보다 isolation 강화
- lightweight (OS specific library들이 포함되지 않기 때문에 container image에 비해 binary의 크기가 매우 작음)

### **Cons?**
- 지원하는 언어가 제한적 (C, C++, Rust)
- OS접근 방식이 다르다: networking, multithread 등
	- cgroups 같은 인터페이스 X
- 산업 표준 관리 & orchestration 도구의 부재


## Reference
- https://alibaba-cloud.medium.com/using-webassembly-and-kubernetes-in-combination-7553e54ea501
- https://dongwoo.blog/2017/06/06/%eb%b2%88%ec%97%ad-%ec%96%b4%ec%85%88%eb%b8%94%eb%a6%ac-%ec%a7%91%ec%a4%91-%ec%bd%94%ec%8a%a4/