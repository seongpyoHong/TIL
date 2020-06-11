### Peer-Finder
MongoDB ReplicaSet을 구축하는 과정 중, Primary Node에 접속하여 Secondary Node의 정보를 추가해줘야 하는 상황 발생

StatefulSet의 생성 이후 수동으로 해줄 수도 있지만, StatefulSet의 Replica 수의 변화를 동적으로 탐지하여 자동으로 등록해주기 위해 현재 Stateful의 Peer들을 알 수 있는 솔루션이 필요

`Peer-Finder`는 Go로 작성되어 있으며, 실행파일의 인자로 `on-start.sh`, `service`,`namespace`를 받는다. 
```
> peer-finder -on-start=<on-start.sh path> -service=<service-name> -namespace=<namespace-name>
```

#### 실행 결과
```
2020/06/11 17:17:45 Peer list updated
was []
now [mongodb-0.mongodb.default.svc.cluster.local mongodb-1.mongodb.default.svc.cluster.local mongodb-2.mongodb.default.svc.cluster.local]
2020/06/11 17:17:45 execing: on-start.sh with stdin: 
mongodb-0.mongodb.default.svc.cluster.local
mongodb-1.mongodb.default.svc.cluster.local
mongodb-2.mongodb.default.svc.cluster.local
```

peer-finder를 init-container에 수행하여 peer의 목록을 찾고, 찾은 peer 목록을 Persistence Volume에 유지하는 방식으로 StatefulSet Replica의 변경에도 동적 설정 가능
