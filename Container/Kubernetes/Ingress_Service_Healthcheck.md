### Service Health Check in Ingress
GKE를 사용하면서 Ingress를 사용하면서 `/kibana` 경로로 등록된 Kibana Service가 정상적으로 동작하고 있음에도 불구하고 Ingress가 읽어오지 못하는 상황이 발생했다.

#### 원인
Kibana Deployment에 등록된 Readiness Probe 정책은 다음과 같다.
```yaml
readinessProbe:
    httpGet:
    path: /
    port: 5601
    timeoutSeconds: 1
    successThreshold: 1
```

이 때, Ingress가 요청하게 되는 경로는 `ingress 경로 + Readiness Probe의 경로`이기 때문에 `/kibana` 경로로 서비스 Health Check를 수행하게 된다.

하지만, Kibana의 Health Check 경로는 `/app/kibana` 이기 때문에 Ingress는 Kibana 서비스가 비정상 상태라고 판단한다.

#### 해결방법
Kibana의 `server.basePath=/kibana`로 변경하여 해결