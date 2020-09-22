### Envoy?
c++로 작성된 고성능 L7 프록시

> L7 프록시는 L4 프록시 보다 정교한 조작이 가능하다. 하지만 이 때문에 속도면에서 L4 프록시보다 느릴 수 밖에 없는데 Envoy는 L7 프록시의 속도 이슈를 해결하기 위해 성능 최적화를 목표로 개발되었다.

Envoy의 특징은 다음과 같다.

1. 고도로 최적화된 내부 서비스 프록시
2. L3/L4애서 수신된 TCP 메시지에 대해 다양한 작업 수행
3. L7 프록시 작업 수행 가능
4. HTTP/2, gRPC 지원


#### 작동 방식
> Downstream ⇒ Listener ⇒ Filter ⇒ Cluster(Upstream)

- Downstream : Envoy에게 요청을 보내는 호스트
- Listener : Downstream에게 요청을 받는 부분
TCP 기반 리스너만 지원한다. 만약 다수의 Downstream에게 요청을 받아야 한다면 여러 Listener로 구성된 단일 Envoy 인스턴스를 실행하는 것이 권장된다.
- Filter : 수신된 메시지에 대해 라우팅, 프로토콜 변환, 통계 생성과 같은 다양한 작업을 수행하는 부분
- Upstream : Envoy가 요청을 보내는 호스트
- Cluster : Upstream Group

### Envoy 실행 (Docker)
Docker Image를 통해 Envoy를 쉽게 실행할 수 있다.
먼저, envoy 이미지를 pull 받는다.
> latest tag가 없으니 직접 tag를 지정해야한다. tag는 envoy document에 있는 태그를 사용하였다.
https://www.envoyproxy.io/docs/envoy/latest/start/start

#### Image Pull
```bash
> docker pull envoyproxy/envoy-dev:f63c27ce1c19bbe4daf991542a9a4f352b36e180
```

먼저, envoy 실행에 사용되는 `envoy.yaml`을 확인해보면 다음과 같다.
```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.google.com
                  cluster: service_google
          http_filters:
          - name: envoy.filters.http.router
  clusters:
  - name: service_google
    connect_timeout: 30s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_google
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.google.com
                port_value: 443
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.api.v2.auth.UpstreamTlsContext
        sni: www.google.com
```

하나씩 확인해보자.
- admin
envoy admin 페이지 설정으로 address를 통해 접속 가능한 IP를, port를 통해 admin page의 port를 의미한다.

> docker를 통해 실행할 경우, `127.0.0.1`은 bridge network의 localhost를 의미하기 때문에 모든 IP를 허용하는 `0.0.0.0`로 변경한다.

- listeners
리스너가 수신할 포트와 주소에 대한 설정값

- filter_chains
    수신한 메세지를 처리할 filter에 대한 설정값으로 기본 설정값은 `/` 경로로 가는 모든 요청을 service_google로 라우팅한다.
    
- clusters
envoy가 요청을 보낼 호스트로 기본 설정값은 google.com으로 라우팅 되도록 설정되어 있다.


#### Envoy Container 실행
위에서 적용했던 admin 변경 사항을 적용한 설정파일을 마운트하여 컨테이너를 실행한다.
```bash
> docker run --rm -d -p 10000:10000 -p 9901:9901 -v /Users/seongpyo/workspace/study/envoy:/etc/envoy/ envoyproxy/envoy-dev:f63c27ce1c19bbe4daf991542a9a4f352b36e180
```

#### 실행화면
- localhost:10000 접속
![](https://images.velog.io/images/sphong0417/post/0876d108-3f5b-4174-8314-6403d354f888/image.png)

- localhost:9901 (Envoy Admin) 접속
![Envoy Admin](https://images.velog.io/images/sphong0417/post/b992ec78-2b32-450a-9759-13ae7503927e/image.png)


Service Mesh에서 Envoy는 사이드카 프록시로 사용된다. 이후에는 Envoy를 관리하는 Istio에 대해 알아보자.
