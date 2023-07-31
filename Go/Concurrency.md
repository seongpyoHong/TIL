### 동시성
Go의 동시성 프로그래밍은 CSP(Communicating Sequential Process)에 기반한다.

동시성은 병렬성이 아니다. (동시성 코드가 병렬적으로 실행되는지 여부는 하드웨어 및 알고리즘이 허용하는지 여부에 따르다.)
- 암달의 법칙: 반드시 순차처리를 해야하는 부분이 존재하기 때문에 병렬 처리를 통해 성능이 더이상 향상되지 않는 한계가 존재한다.
- 동시성을 통해 값을 전달핮는 오버헤드로 인해 더 느려질 수도 있다.

### Goroutine
고루틴은 Go Runtime에서 관리하는 프로세스
- Go 런타임은 여러 thread를 생성하고 프로그램을 실행하기 위한 단일 고루틴을 시작
- Go 런타임 스케줄러가 자동으로 스레드을 할당

**운영체제에서 제공하는 프로세스 & 스레드와 비교했을 때 장점**
- 운영체제 레벨에서 자원을 생성하지 않기 때문에 스레드 생성보다 오버헤드가 작다
- 초기 스택크기가 스레드의 스택 크기보다 작고, 필요에 따라 늘릴 수도 있기 때문에 메모리 효율이 증가한다.
- context switchig이 프로세스 내부에서 일어나기 때문에 kernel space를 거치는 쓰레드 context swtching보다 빠르다.
- 스케줄러 최적화가 가능하다. (gophercon 영상 확인)

모든 함수는 고루틴으로 실행될 수 있지만, 비즈니스 로직을 감싸는 클로저를 고루틴으로 실행하는 것이 관례
- 채널의 외부에서 값을 읽어 비즈니스 로직에 전달 => 함수의 결과가 다른 채널에서 다시 기록되도록 하여 책임을 분리

```go
func doSomething(target int) {
	// do something with target
}

func DoconcurrencySomething(in <-chan int, out chan<- int) {
	go func() {
		for v := range in {
			result := doSomething(v)
			out <- result
		}
	}()
}
```


### Channel
고루틴은 채널을 통해 통신한다.
- zero value는 nil

#### channel 연산자
- `<-` : 채널의 데이터를 읽고 쓰기
```go
// ch에서 값을 읽어 a에 할당
a := <-ch

// b를 ch에 write
ch <- b
```

- 채녈에 쓰여진 각 값은 한 번에 하나씩 읽을 수 있다.
	- 여러개의 goroutine에서 채널의 데이터를 읽으려고 한다면 데이터 당 하나의 goroutine만 읽을 수 있다.
- 하나의 고루틴으로 같은 채널을 읽고 쓰기를 하는 것은 드물다.
	- read only channel: ch <-chan int
	- write only channel: ch chan<- int

- 기본적으로 채널에는 버퍼가 없다.
	- 버퍼가 없는 채널에 데이터를 쓰면 다른 고루틴에서 같은 채널을 읽을 때 까지 쓰기 작업을 하던 고루틴은 일시 중지된다.
	- 버퍼가 있는 채널을 만들 수도 있다. (`ch := make(chan int, 10)`)
		- 버퍼가 가득차면 쓰기 작업을 하던 고루틴은 일시정지


#### read data from channel
for-range loop를 이용하여 채널의 값을 읽을 수 있다.
- 채널이 닫히기 전까지
```go
for v := range ch {
	fmt.Println(v)
}
```


#### Close channel
```go
close(ch)
```
- 닫힌 채널에 쓰기를 시도하거나 다시 close를 시도하면 panic을 발생시킨다.
	- 여러 고루틴에서 같은 채널에 쓰기를 시도하는 경우, close의 중복 및 닫힌 채널에 쓰기 작업을 함으로 panic이 발생할 가능성이 있다.
	- 이를 위해 sync.WaitGroup을 사용하여 해결할 수 있다.
- 닫힌 채널에 읽기를 시도하면 항상 성공한다. 만약 버퍼에 값이 남아있으면 남아있는 값을 순차적으로 반환하고 비어있다면 해당 채널의 zero value를 리턴한다.
- 다음과 같이 channel이 닫혔는지 여부를 확인할 수 있다.
```go
v, ok := <-ch
```

### sync.WaitGroup
waitGroup의 함수는 다음과 같다.
- func (wg * WaitGroup) Add(delta int): waitGroup에 고루틴 개수 추가
- func (wg * WaitGroup) Done(): 고루틴이 끝났음을 알려줄 떄 사용
- func (wg * WaitGroup) Wait(): 모든 고루틴이 끝날 때까지 기다림

> Add와 Done의 개수는 반드시 같아야 한다. (그렇지 않으면 panic 발생)

#### Select
두 개의 동시성 연산을 수행해야 한다면 어떤 것을 먼저 실행해야 할지 구분이 필요하다. select 키워드는 여러 채널 중 하나에 읽기를 하거나 쓰기를 할 수 있는 고루틴을 허용한다.
```go
select {
case v := <-ch:
	fmt.Println(v)
case v := <-ch2:
	fmt.Println(v)
case ch3 <- x:
	fmt.Println("wrote", x)
case <-ch4:
	fmt.Println("got value on ch4, but ignore(not print)")
}
```

- 동작 방식
	- 진행 가능한 여러 case 중 하나를 임의로 선택 (모두 동시에 확인하기 때문에 기아 문제 해결)
	- 모든 고루틴이 deadlock 상태에 빠진다면 go runtime은 해당 프로그램을 제거한다.


- Deadlock 예시
```go
package main

import (
	"fmt"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()
	v := 2
	ch2 <- v
	v2 := <-ch1
	fmt.Println(v, v2)
}
```
- main 고루틴은 ch2에서 데이터를 읽어야지만 진행되고, 생성된 고루틴은 ch1에서 데이터를 읽어야 진행되기 때문에 deadlock이 발생
- select를 사용하면 deadlock 해결 가능
```go
package main

import (
	"fmt"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()
	v := 2
	var v2 int
	select {
		case ch2 <-v:
		case v2 = <-ch1
	}
	fmt.Println(v, v2)
}

// result
2 1
```

select는 여러 채널을 통한 통신을 담당하기 때문에 for loop에 임베딩하여 사용하는 것이 일반적인 패턴 (for-select라고도 부름)
```go
for {
	select {
		case <-done:
			return
		case v := <-ch:
			fmt.Println(v)
	}
}
```

- select 문도 default 조건이 존재: 읽고 쓸 수 있는 채널이 하나도 없는 경우에 해당됨

---

## 동시성 사례 및 패턴
#### 동시성 없이 API 유지
동시성은 구현 세부 내용이기 때문에 API 설계에서는 채널 및 mutex와 같은 동시성 관련 로직을 노출해서는 안된다.
- 타임 아웃을 처리하는 time.After와 같이 채널의 API의 일부가 되는 경우도 물론 존재한다.

#### 고루틴, for loop, 가변 변수
대부분의 경우, 고루틴으로 사용하는 클로저에는 파라미터를 사용하지 않지만 필요한 경우가 있다.
- 값이 바뀔 수 있는 변수에 의존하는 고루틴이라면 해당 값을 고루틴으로 반드시 넘겨줘야한다.
```go
// as-is
a := []int{2, 4, 6, 8, 10}
for _, v := range a {
	go func() {
		ch <- v * 2
	}()
}

// result
20
20
20
20
20


// to-be
for _, v := range a {
	go func(val int) {
		ch <- val * 2
	}(v)
}
```

#### 고루틴 정리
고루틴이 종료되지 않으면 스케줄러는 고루틴이 다시 사용되는지 알 수 없기 때문에 계속해서 스케줄링하고 이로 인해 프로그램이 느려지게 된다.

#### Done 채널 패턴
Done 채널 패턴은 처리를 종료해야 하는 시점을 고루틴에게 알리는 방법을 제공한다.
```go
func searchData(s string, searchers []funcs(string) []string) []string {
	done := make(chan struct{})
	result := make(chan []string)
	for _, searcher := range searchers {
		go func(searcher func(string) []string) {
			select {
				case result <- searcher(s):
				case <-done:
			}
		}(searcher)
	}
	r := <-result
	close(done)
	return r
}
```

- done 채널은 close 될 때까지 읽기는 일시중지이고 close되면 읽기 연산에 대해 zero value가 반환된다.
=> done 채널을 통해 goroutine에게 종료될 시점을 알려준다.

### 취소 가능한 Context
호출 스택의 이전 함수에서 가져온 context를 기반으로 고루틴을 종료하고 싶을 때 취소가 가능한 context을 사용한다.
- 각각 다른 HTTP 서비스를 호출하는 여러 고루틴을 생성하는 요청이 있을 경우, 하나의 서비스가 오류를 반환하면 다른 고루틴을 계속 처리할 필요가 없다.
- `context.WithCancel` 을 통해 취소 가능한 context를 생성한다.
```go
var client = http.Client()

func callBoth(ctx context.Context, errVal string, url1, url2 string) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()
	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		defer wg.Done()
		err := callServer(ctx, "server1", url1)
		if err != nil {
			cancel()
		}
	}()

	go func() {
		defer wg.Done()
		err := callServer(ctx, "server2", url2)
		if err != nil {
		cancel()}
	}()
	wg.Wait()
	fmt.Println("done with both")
}
```
- 취소 가능한 context를 만들 때마다 취소 함수를 항상 호출해줘야한다.
	- 처음 호출된 cancel 이후에는 모두 무시됨
	- 각 callServer에서 에러가 발생하면 cancel을 호출
	- 추가로 defer를 통해 cancel이 최종적으로 수행됨을 보장

#### 취소 함수
채널을 닫을 떄, 직접 반환하는 것보다 클로저를 사용하는 것이 좋다. (추가적인 정리 작업도 실행할 수 있기 때문에)
```go
func doSomething() (<-chan int, func()) {
	ch := make(chan int)
	done := make(chan int)
	cancel := func() {
		close(done)
	}

	return ch, cancel
}

func main() {
	ch, cancel := doSomething()
	//...
	cancel()
}
 ```

#### 버퍼가 있는 채널 사용 시점?
- 얼마나 많은 고루틴이 실행될지 알고 있을 떄, 실행 중인 고루틴의 개수를 제한할 때
- 대기 중인 작업의 양을 관리할 때
- Backpressure
```go
// Backpressure 예제
type Pressure struct {
	ch chan struct{}
}

func NewPressure(limit int) *Pressure {
	ch := make(chan struct{}, limit)
	for i := 0; i < limit; i++ {
		ch <- struct{}{}
	}
	return &Pressure{
		ch: ch,
	}
}

func (p *Pressure) Process(f func()) error {
	select {
		case <-p.ch:
			f()
			pg.ch <- struct{}{}
			return nil
		default:
			return errors.New("no capacity")
	}
}

```

#### Timeout
`time.After(duration)` 은 duration마다 값을 쓰는 channel을 리턴한다.

#### 정확히 한 번만 코드 실행
`sync.Once` 를 사용
```go
var parser Parser
var once sync.Once

func Parse(raw string) string {
	once.Do(func() {
		parse = initParser()
	})
}
```

#### 채널 vs Mutex
golang에서 채널과 select를 사용하는 이유는 데이터의 흐름이 mutex를 사용하는 것보다 명확해진다.

때로는 뮤텍스를 사용하는 것이 더 명확한 경우도 존재 
- 고루틴을 조정하거나 고루틴에 의해 변경되는 값을 추적하는 경우에는 채널 사용
- 구조체에 항목을 공유하여 접근하는 경우는 뮤텍스 사용
- 채널 사용시 성능의 문제가 있는 경우, 뮤텍스로 구현 시도

- sync.Lock, sync.Unlock 사용
	- Lock: 임계 영역에 현재 다른 고루틴이 머무는 한 현재 고루틴을 일시 중지
	- 임계영역이 해제되면 현재 고루틴이 잠금을 획득하고 임계 영역의 코드가 실행됨
	- Unlock: 임계 영역의 마지막을 표시
- sync.RWMutext
	- 하나의 작성자가 임계영역 내에 있을 수 있지만 읽기 잠금은 공유됨 => 한 번의 임계 영역에 여러 독자가 존재 가능
	- 쓰기: Lock / Unlock
	- 읽기: RLock, RUnlock

- Channel을 사용하는 버전: Read 시 한 번에 하나의 읽기만 허용하고 번거로움
```go
func manager(in <-chan func(map[string]int, done <-chan struct{})) {
	board := map[string]int{}
	for {
		select {
			case <-done:
				return
			case f := <-in:
				f(board)
		}
	}
}

type ChannelManager chan func(map[string]int)

func NewChannelManager() (ChannelManager, func()) {
	ch := make(ChannelManager)
	done := make(chan struct{})
	go manager(ChannelManager, done)
	return ch, func() {
		close(done)
	}
}

func (cm ChannelManager) Update(name string, value int) {
	cm <-func(m map[string]int) {
		m[name] = value
	}
}

func (cm ChannelManager) Read(name string) (int, bool) {
	var out int
	var ok bool
	done := make(chan struct{})
	cm <-func(m map[string]int) {
		out, ok := m[name]
		close(done)
	}
	<-done
	return out, ok
}
```

- Mutex를 사용하는 버전
```go
type MutexManager struct {
	mutex sync.RWMutex
	board map[string]int
}

func NewMutexManager() *MutexManager {
	return &MutexManager{
		board: map[string]int{}	
	}
}

func (mm *MutexManager) Update(name string, value int) {
	mm.mutex.Lock()
	defer mm.mutex.Unlock()
	mm.board[name] = value
}

func (mm *MutexManager) Read(name string) (int, bool) {
	mm.mutex.RLock()
	defer mm.mutex.RUnlock()
	val, ok := mm.board[name]
	return val, ok
}
```

