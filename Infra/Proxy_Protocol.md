## Proxy Protocol
proxy protocol은 NAT 또는 TCP 프록시의 여러 계층에 걸쳐 클라이언트 주소와 같은 연결 정보를 전송할 수 있는 편리한 방법을 제공하는 프로토콜이다.
일반적으로 TCP connection이 proxy들을 통해 연속되어 있는 경우, source/destination 주소, 포트 정보 등 origin TCP 커넥션이 가지고 있는 정보의 손실이 발생하게 된다.
HTTP에서는 `X-Forwared-For` header를 통해 origin source address 정보를 전달할 수도 있다. 

하지만 이런 방법은 HTTP에 대한 이해가 동반되어야 한다.

#### dum proxy?

protocol에 상관없는 데이터를 처리하는 proxy (e.g. Stunnel, Stud)

이러한 프록시가 haproxy와 같은 다른 프록시와 결합될 때 발생하는 문제는 상위 수준의 프로토콜을 사용하도록 프록시를 조정하는 것 

- 예시 
```
      +--------+      HTTP                      :80 +----------+
      | client |  --------------------------------> |          |
      |        |                                    | haproxy, |
      +--------+             +---------+            |  1 or 2  |
     /        /     HTTPS    | stunnel |  HTTP  :81 | listening|
    <________/    ---------> | (server | ---------> |  ports   |
                             |  mode)  |            |          |
                             +---------+            +----------+

```
stunnel은 들어오는 각 연결의 첫 번째 HTTP 요청에 X-Forwarded-For 헤더를 삽입할 수 있는 패치 적용 가능
하지만, Haproxy는 연결이 Stunnel에서 올 때 다른 연결을 추가할 수 없으므로 서버로부터 연결을 숨길 수 있습니다.

ha proxy가 클라이언트에 대해 keep-alive로 실행될 때 문제
=> 스턴넬 패치는 X-Forwarded-For 헤더를 각 연결의 첫 번째 요청에만 추가하기 때문에 이후의 모든 요청에는 해당 헤더 X. 

해결 방법은 패치 개선(전달된 모든 데이터(Content-Length 또는 Transfer-Encoding 포함)를 지원하거나 전송하지 않고 데이터를 알리는 HEAD와 같은 특수한 방법을 처리하는 것)
이렇게 되면 Stunnel에서 전체 HTTP 스택을 구현해야 하고 이는 더이상 dummy proxy의 목적에 맞지 않는다.

다른 접근 방법은 각 연결을 다른 쪽 연결의 특성을 보고하는 헤더로 추가하는 방법
=> 구현도 쉽고, protocol에 종속 X => Proxy Protocol


----------- 2021-08-10 / TODO: proxy protocol + 어떤 장점을 가지는지 


### 참고자료
- https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt
- https://www.haproxy.com/blog/using-haproxy-with-the-proxy-protocol-to-better-secure-your-database/
