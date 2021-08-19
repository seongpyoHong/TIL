## Range Expression
golang에서는 range expression을 통해 다른 언어에서의 for each 반복문을 수행할 수 있다.

하지만 주의할 점은 range expression에서 value는 실제 값이 아닌 복사된 값을 의미한다는 점이다.

아래와 같은 예시를 들어보면, 
```golang
arr := make([]int, 3)
arr[0], arr[1], arr[2] = 1, 2, 3

for idx, value := range arr {
    fmt.Println(&arr[idx], " vs ", &value)
}
```
원본 slice에 있는 원소의 주소와 for 문 안에서의 value 주소가 다른 것을 확인할 수 있다.
```shell
0xc00008a000  vs  0xc00008c000
0xc00008a008  vs  0xc00008c000
0xc00008a010  vs  0xc00008c000
```

### 참고자료
- https://golang.org/ref/spec#For_statements
- https://stackoverflow.com/questions/15945030/change-values-while-iterating
