### Background
- Envoy Sidecar에서 TLS Termination 이후 Application Container로 HTTP 요청 전달하는 구성
- 이 때, Application Container는 X-Forwarded-For 헤더를 사용
- 해당 Pod에 물려있는 LB는 L4LB이기 떄문에 X-Forwarded-For 헤더 값이 따로 없음
- Envoy에서 `X-Forwarded-For`에 Client IP를 넣어주기 위해 `use_remote_address` 을 True로 설정
	- `use_remote_address` 가 true면 envoy에서 실제 client IP를 `X-Forwarded-For`에 추가
- 하지만, Service가 `LoadBalancer` 타입이고, `externalTrafficPolicy`가 `Cluster` 여서 Envoy가 Client IP를 SNAT로 인해 변경된 IP로 착각함
- `externalTrafficPolicy`를 `Local`로 수정하여 실제 Client IP가 유지되도록 설정
	- 사용하고 있는 L4LB는 IP 기반 Hashing 알고리즘으로 LoadBalancing을 하고 있어 트래픽이 특정 Node에 쏠릴 가능성이 존재 => TODO

### Reference
- `use_remote_address`: https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-use-remote-address
- `externalTrafficPolicy`: https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
