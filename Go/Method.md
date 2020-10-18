go에는 클래스라는 개념이 없다. 자바나 C++에서 클래스가 멤버변수와 메소드로 하던 행위들을 `Struct`와 Receiver라는 부분이 포함된 `메소드`를 통해 수행할 수 있다.

먼저, `struct`를 통해 필드를 정의할 수 있다.

```go
type album struct {
	title string
	sontCnt int
	playtimes map[string]int
}
```

이 구조체에 대한 함수는 receiver라는 특별한 부분을 갖는 `메서드`라는 것으로 표현할 수 있다. **알아둬야할 것은 go에서 함수와 메서드는 다른 의미로 사용된다는 점이다.** 

### 함수 vs 메소드

함수와 메소드의 차이점에 대해 간략하게 정리하면 다음과 같다. 

- 함수

    Parameter, return, body를 명시함으로써 선언

    1. Stateless
- 메소드

    추가적으로 Receiver라는 것을 명시함으로써 선언

    1. Method Chaining
    2. Stateful

참고자료 : [https://velog.io/@kameals/Golang-functions-vs-methods](https://velog.io/@kameals/Golang-functions-vs-methods)

`method`를 통해 struct에 대한 함수를 호출할 수 있다.

```go
//struct
type album struct {
	title string
	playtimes map[string]int
}

//initializer
func newAlbum() *album {
	a := album{}
	a.songs = map[int]string{}
	a.songs[1] = "song1"
	return &a
}

//method
func (a album) totalSongCount() int {
	return len(a.songs)
}

func main() {
	a := newAlbum()
	print(a.totalCount())
}

///result : 1
```

Receiver 부분을 포인터 값으로 받게 되면 method안에서 수행한 작업이 struct의 값에 영향을 미치게 된다.

```go
func (a *album) addSong(name string) {
	curIdx := len(a.songs)
	a.songs[curIdx+1] = name
}

func main() {
	a := newAlbum()
	println(a.totalCount())
	a.addSong("added")
	println(a.totalCount())
}
```
