### Pod 정상 종료 보장

컨테이너를 종료하는 경우, kubelet은 SIGTERM을 전송한다. SIGTERM을 일정 시간동안 정상적으로 처리하지 못하는 경우 SIGKILL을 발생시켜 종료한다. Graceful Shutdown을 구현하기 어려운 경우, LifeCycle Hook인 PreStop Hook을 사용하여 컨테이너 종료 전 임의의 작업을 수행하도록 할 수 있다. 

예를 들면, Graylog는 Cluster 종료 간 메세지 손실을 막기 위해 Input을 먼저 close하고, 모든 buffer 및 캐싱된 메세지의 처리를 시도하는 shutdown API를 제공한다. 

```json
POST /system/shutdown/shutdown
```

Prestop Hook에 해당 API를 호출하도록 등록하여 종료 시 메세지 손실을 방지할 수 있으며, SIGTERM을 정상적으로 처리하지 못하는 경우에 대비할 수도 있다.

```yaml
lifecycle:
  preStop:
      exec:
          command:
            - bash
            - -ec
            - |
                curl -XPOST -sS \
                    -u "{{ .Values.graylog.rootUserName }}:${GRAYLOG_PASSWORD_SHA2}" \
                    -H "X-Requested-By: admin \
                        "http://localhost:9000/api/system/shutdown/shutdown"
```
