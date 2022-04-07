Ref: https://go.dev/tour/concurrency/4

recieved channel에 range를 사용하면 close 될 때까지 값을 받아온다.

```golang
func main() {
	c := make(chan int, 5)
	go func(){
    defer close(c)
    for i := 0; i < 5; i++ {
      c <- i
    }
  }()
  
	for i := range c {
		fmt.Println(i)
	}
}
```

output
```
0
1
2
3
4
```
