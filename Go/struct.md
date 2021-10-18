## Struct
구조체 내부의 struct embeding을 통해 내부 struct의 field를 바로 참조할 수 있다.
```golang
type Person struct {
  name string
}

type Student struct {
  Person
  grade int
}

var s Student
// 바로 참조 가능
s.name = "sphong" 
```

=> field 뿐만 아니라 method도 참조 => composition에서도 활용 가능

## JSON
Golang에서는 `enconding/json`과 같은 여러 포맷으로의 encoding/decoding 지원
- Marshal : data for Golang => JSON
- Unmarshal : JSON => data for Golang
  
  필요한 field만 복호화 가능 (Struct에 정의된 field만 복호화됨)

field tag를 통해 JSON 객체의 필드명 지정
```
type A struct {
  aa `json:"aa"`
  bb `json:"bb"`
}
```
