### Background
Index Mapping에 맞지 않는 로그가 들어왔을 때 어떻게 처리할지에 대해 고민 중이었다.
- Dead Letter Queue / Index?

Opensearch를 백엔드로 쓰고 있는 상용 솔루션인 Logzio를 참고해보니, Mapping Error가 발생했을 경우 해당 로그를 다른 필드로 저장하는 방법을 사용했다.
- 에러난 로그를 다른 필드에 저장
- index-failed-reason 필드에 실패한 원인을 저장
- type에 logzio-index-failure로 저장하여 정상 로그와 구분


### Reference
https://docs.logz.io/docs/user-guide/explore/troubleshooting/invalid-log-errors/`