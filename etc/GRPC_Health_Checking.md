# gRPC Health Check

### gRPC Health Checking Protocol
gRPC server의 health check란 server가 rpc를 핸들링할 준비가 되어있는지 검증하는 과정
Client to server health check는 `point-to-point` 나 `control system`을 통해 수행됨
	- server가 어떠한 이유로 request를 처리할 준비가 되어있지 않다.
	- client는 정해진 시간동안 response를 받지 못하거나, unhealthy response를 받게 됨

gRPC 서비스는 client-to-server 시나리오와 LB와 같은 control system을 커버할 수 있는  health checking 메카니즘을 사용
	1. Health check도 rpc와 동일한 format (이것도 역시 gRPC)
	2. 각 서비스 상태를 나타내기 위한 충분한 semantic을 포함하고 있어야 함
	3. 기존 서비스에서  재사용 가능하며, server가 health check 서비스 접근에 대한 모든 권한을 가진다.

### Service Definition
```proto
syntax = "proto3";

package grpc.health.v1;

message HealthCheckRequest {
  string service = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
    SERVICE_UNKNOWN = 3;  // Used only by the Watch method.
  }
  ServingStatus status = 1;
}

service Health {
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);

  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
}
```

- Client는 `Check` 메소드를 통해 server의 상태를 체크
- Server는 모든 서비스를 메뉴얼하게 등록, 각 상태를 설정해줘야 한다.
- service 이름이 registry에 존재한다면, response는 `OK` status와 `SERVING`, `NOT_SERVING`으로 설정된 status field를 반환
- service 이름이 registry에 존재하지 않는다며 , `NOT_FOUND` gRPC status를 반환
- Client는 rpc가 정해진 시간 안에 종료되지 않는다면 server를 `unhealth` 로 정의할 수 있다.
- Client는 상태를 질의하기 위한 service 이름을 정의할 수 있다. (e.g. `grpc.health.v1.Health`)
- Client는 `Watch`  method를 통해 streaming health check를 수행할 수 있다. Server는 현재 status를 즉시 리턴하고, 상태가 바뀔 때마다 새로운 메세지를 전송한다.

### gRPC-health-probe
Kuberenets에서 제공하는 probe로는 roc의 health check를 할 수 없기 때문에, gRPC Health Checking Protocol을 구현한 service를 probe할 수 있는 command-line tool을 제공한다.
이를 kubernetes probe에서 exec를 실행함으로써 rpc probe를 수행할 수 있다.

### Kubernetes gRPC liveness probe [v1.23 alpha]

Kubernetes 상에 올라간 application이 gRPC Health Checking Protocol을 구현하고 있다면 `grpc` livenessProbe를 사용할 수 있다.
(아직 alpha version)

## References 
- https://github.com/grpc/grpc/blob/master/doc/health-checking.md



