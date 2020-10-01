익명함수란 함수명을 갖지 않는 함수를 의미한다.

```go
func testAnonymousFunction() int {
	sum := func(x int, y int) int {
		return x + y
	}
	return sum(1,2)
}
```


Go에서 함수는 **일급객체**로 취급되기 때문에 함수의 파라미터 및 리턴값으로 사용될 수 있다.

```go
func testAnonymousFunction() int {
	sum := func(x int, y int) int {
		return x + y
	}
	return calcSumAndMultiply(sum, 1, 2, 3)
}

type calculator func(int, int) int

func calcSumAndMultiply(sum calculator, x int, y int, z int) int {
	return sum(x, y) * z
}
```

참고로, go는 `type`을 통해 **Delegate** 기능을 제공한다.
