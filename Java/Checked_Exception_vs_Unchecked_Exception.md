### 예외 처리

**예외를 사용하는 이유**

- 문서화 :  시그니처 자체에 예외를 지원한다.
- 관심사 분리 : 비즈니스 로직과 예외 회복이 try-catch 블록으로 구분된다.

**자바는 확인된 예외와 미확인 예외 두 종류의 예외를 지원한다.**

- 확인된 예외

    방지할 수 없는 잘못된 상황에  대한 예외로 처리 흐름상 발생할 수 있는 예외를 처리하기 위헤 사용된다.  처리코드가 있는지 없는지 컴파일러가 체크할 수 있다.

- 미확인 예외 (Unchecked Exception == Runtime Exception)

    Code를 잘못 만들어 생기는 예외, 컴파일에서는 문제가 없으나 실행시 문제가 발생한다. Runtime Exception의 하위 예외들은 try-catch보다는 이런 예외가 발생하지 않도록 프로그램을 변경하는 것이 옳은 방법이다.

**주의점**

- 예외를 무시하지 않는다.
- 특정 구현에 종속된 예외를 사용하지 않는다.
- 예외를 제어 흐름에 사용하지 않는다.

**Exception hierarchy**

```
 Object
   |
Throwable
   |
Exception ------------------------------------ CheckedException
          |- ClassNotFoundException
          |
          |- IOExceton
          |
          ----------------------------------- UncheckedException
          |- RuntimeException 
```
