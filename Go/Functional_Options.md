## Functional Options
Builder Pattern과 같이 여러 optional params를 받아 객체를 생성할 니즈가 존재할 경우, 사용할 수 있는 패턴

Golang에서는 exception이 없기 떄문에 error를 리턴하는 방법밖에 없다. 따라서 builder pattern을 통한 method chaining을 이용하기에 제한이 있다.
[blog](https://www.calhoun.io/using-functional-options-instead-of-method-chaining-in-go/) 예제를 참고해보면

- 아래와 같이 매번 error를 체크해야하기 때문에 method chaining 제한
```golang
var db *gorm.DB
var err error
db, err = db.Where("id = ?", 123)
if err != nil { ... }
db, err = db.Where("email = ?", "jon@calhoun.io")
if err != nil { ... }
// ... and so on
```

- 아래와 같이 부가적인 Error 필드를 부착?하는 방법을 사용하면 method chaining을 사용할 수는 있지만, 사용자가 Error field에 대해 알아야하고, chaining method 중 어디에서 error가 발생하는지 명확하지 않다.
```golang
db, _ := gorm.Open(...)
var user User
err := db.Where("email = ?", "jon@calhoun.io").
  Where("age >= ?", 18).
  First(&user).
  Error
```

이거를 보다 쉽게 해결할 수 있는 방법이 `Functional Options`.
```golang
type QueryOption func(db *gorm.DB) (*gorm.DB, error)

func Where(query interface{}, args ...interface{}) QueryOption {
  return func(db *gorm.DB) (*gorm.DB, error) {
    ret := db.Where(query, args)
    return ret, ret.Error
  }
}

func (db *DB) First(out interface{}, opts ...QueryOption) error {
  // Get the GORM DB
  gdb := db.DB
  var err error = nil
  // Apply all the options
  for _, opt := range opts {
    gdb, err = opt(gdb)
    if err != nil {
      return err
    }
  }
  // Execute the `First` method and check for errors
  if err := gdb.First(out).Error; err != nil {
    return err
  }
```

각 옵션에 대해서 error 체크를 할 수 있기 때문에 사용자가 Error 타입처럼 API의 세부정보에 대해서 알 필요가 없어진다.
또한, Builder Pattern이 가지고 있는 장점인 API의 유연성(변수 추가에 능하고, 필요한 데이터만 설정 가능)도 확보할 수 있다.


### 참고자료
- https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis (시초?)
- https://www.calhoun.io/using-functional-options-instead-of-method-chaining-in-go/
