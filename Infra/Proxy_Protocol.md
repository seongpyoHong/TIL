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
### Proxy Protocol?
client-server가 direct로 연결되었을 때 얻을 수 있던 정보들을 proxy를 사용하면서 얻을 수 있게 하기위한 protocol
- v1 : human readable header
- v2 : binary header

#### Architectural benefits
1. multiple layer의 구조일 경우, transparent proxy의 경우에는 방화벽에서 client-ip를 처리해야하 하기 때문에 처리해야할 작업의 복잡도가 증가하지만, proxy protocol을 사용하게 되면 이전 proxy의 ip가 방화벽을 통과하기 때문에 쉽게 적용할 수 있다.
```
         Internet
          ,---.                     | client to PX1:
         (  X  )                    | native protocol
          `---'                     |
            |                       V
         +--+--+      +-----+
         | FW1 |------| PX1 |
         +--+--+      +-----+       | PX1 to PX2: PROXY + native
            |                       V
         +--+--+      +-----+
         | FW2 |------| PX2 |
         +--+--+      +-----+       | PX2 to SRV: PROXY + native
            |                       V
         +--+--+
         | SRV |
         +-----+
```

2. Mutiple Proxy
여러 Data Center가 있는 아래와 같은 경우, 각 DC는 L3 LB에서 처리하는 VIP를 사용하게 된다. L3 LB는 트래픽을 L7 SSL/cache offloader farm으로 라우팅하며 해당 farm은 로컬 서버간의 로드 밸런싱을 수행한다.

이 때, VIP DC에 종속적이게 되는데 클라이언트는 하나의 DC에 종속적이면 안되기 떄문에 L7 Proxy는 인터넷이나 LAN을 통해 접근할 수 있는 다른 DC 서버도 알 수 있어야 한다.
이 때 PROXY protocol을 사용하게 되면 DC간의 트래픽에도 원래 서버 주소를 전달할 수 있다. 하지만 Transparent proxy를 사용하게 되는 경우, 대부분의 L7 proxy는 adress spoofing이 불가능하기 떄문에 사용할 수 없다.
```
                               Internet

            DC1                  DC2                  DC3
           ,---.                ,---.                ,---.
          (  X  )              (  X  )              (  X  )
           `---'                `---'                `---'
             |    +-------+       |    +-------+       |    +-------+
             +----| L3 LB |       +----| L3 LB |       +----| L3 LB |
             |    +-------+       |    +-------+       |    +-------+
       ------+------- ~ ~ ~ ------+------- ~ ~ ~ ------+-------
       |||||   ||||         |||||   ||||         |||||    ||||
      50 SRV   4 PX        50 SRV   4 PX        50 SRV    4 PX
```

### 참고자료
- https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt
- https://www.haproxy.com/blog/using-haproxy-with-the-proxy-protocol-to-better-secure-your-database/
