### Background
- Golang으로 작성한 Application을 K8S에 배포한 후, 성능 테스트를 진행하면서 OOM이 발생
- 찾아보니 `GOMEMLIMIT` 이라는 환경변수 값을 통해서 Golang Heap Memory Usage의 Soft Limit을 정할 수 있다.
- 하지만, `GOMEMLIMIT`  을 적용한 이후에도 동일하게 OOM이 발생했다. 이에 대해 GC의 동작 원리를 파악하여 원인을 찾기 위해 Golang GC 옵션 중 Heap 용량에 영향을 주는 옵션들에 대해 알아보게 되었다.

### Golang GC
- 자세히 읽지는 않았지만 Golang GC는 다른 언어들의 GC에 비해 효울적이라고 한다. (https://tip.golang.org/doc/gc-guide#Understanding_costs)
- `GOGC` : 이전 시점에 사용하고 있는 Heap Memory의 몇 %가 되었을 때 GC을 실행할 것인가?
	- 예를 들어 해당 값이 `100` 이고 이전 시점 Heap 사용량이 2GB이면 현 시점 Heap 사용량이 4GB 일 때 GC가 발생한다.

- Golang 1.19 버전 이전에는 `GOGC` 만 존재했었다. 이는 다음과 같은 문제가 존재한다.
	- `GOGC` 는 이전 시점에 대한 상대 값이기 떄문에 너무 적게 주면 GC가 너무 빈번하게 일어나고, 너무 크게 주면 OOM이 발생할 확률이 높아진다.
	- 최선의 선택은 메모리가 적을 떄는 GC가 적게 발생하면서도 Limit에 근접해지면 GC가 빈번하게 발생하는 것이다.

- 이를 해결하기 위해 Golang 1.19 부터 `GOMEMLIMIT` 이 도입되었다. 
	- `GOMEMLIMIT` 은 Heap 메모리의 Soft Limit으로 GC는 이를 넘지 않기 위해 노력하게 된다.
	- 즉, `GOGC` 값을 이전보다 여유롭게 줘서 Memory를 적게 사용할 떄에는 GC가 덜 발생하게 만들면서도, Memory Limit에 가까워지면 GC를 빈번하게 발생시킬 수 있게 되었다.

- 하지만 말 그대로 Soft Limit이기 떄문에 실제로 해당 값을 넘지 않는 것을 보장하지는 못한다. 나의 경우, 10GB를 할당받은 Pod에 `GOMEMLIMIT` 을 4GB로 설정했음에도 OOM이 발생하였다.

### Reference
https://weaviate.io/blog/gomemlimit-a-game-changer-for-high-memory-applications
