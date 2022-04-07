Ref: https://pkg.go.dev/bufio#Scanner

### Background
`io.Reader`로 넘어오는 값을 `\n`으로 파싱하고, 해당 값을 `json.Unmarshal([]byte, interface{})`에 사용하기 위해 다시 []byte로 바꿔야하는 상황

처음 접근방법: `buf.Read`를 통해서 읽고, 이를 `buf.String()`으로 변환하여 파싱 => 다시 `[]byte`로 형변환

`bufio.Scanner`를 사용하면 아래와 같이 처리 가능
```golang
func main() {
  var ir io.Reader
  /*
  set value
  */
  scanner := bufio.NewScanner(ir)
  for scanner.Scan() {
    var obj Object
    json.Unmarshal(scanner.Bytes(), &obj)
  }
}
```
