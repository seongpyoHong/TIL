### Kakfa Sequential I/O

일반적으로 Disk와 Memory의 I/O 속도를 비교할 때 Memory가 속도가 빠르며, Disk I/O가 느린 이유는 Seek Time 때문이다. 즉, Seek Time에서 걸리는 속도를 줄일 수 있다면 Disk도 Memory와 유사한 성능을 낼 수 있다.

Kafka는 Disk I/O의 병목 구간인 Seek Time을 줄이기 위해 Sequential I/O 라는 방법을 사용한다. 

Kafka 로그 적재의 기본 개념은 `Append Only` 으로 메세지를 추가할 수만 있고 삭제는 불가능하기 때문에 `immutable` 한 특성을 가진다. 이를 통해 Offset으로 읽을 위치를 관리할 수 있게 된다.

- **Sequential Access** : 논리적/물리적 순서를 따라 차례대로 읽어 나가는 방식

    ⇒ Offset을 통해 순차적으로 읽을 수 있다.

- **Random Access** :  레코드 간 논리적, 물리적인 순서를 따르지 않고, 한 블록씩 접근하는 방식

계속해서 Seek 과정이 발생하는 Random Access와 달리 Sequential는 offset을 통해 Seek 과정 없이도 접근할 수 있다. 이를 통해 Disk I/O로 발생하는 Seek Time Overhead를 줄일 수 있다.

**참고 자료**

- [https://medium.com/@sunny_81705/what-makes-apache-kafka-so-fast-71b477dcbf0](https://medium.com/@sunny_81705/what-makes-apache-kafka-so-fast-71b477dcbf0)
- [https://epicdevs.com/17](https://epicdevs.com/17)
- [http://www.gurubee.net/lecture/2394](http://www.gurubee.net/lecture/2394)
