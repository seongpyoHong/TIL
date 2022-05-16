consumer group에 속한 인스턴스는 특정 토픽에 해당하는 파티션을 나눠가짐

instance가 추가되거나 삭제되면(장애) 리밸런싱을 통해 파티션을 다시 나눠가진다.

컨슈머 그룹별로 토픽에 대한 offset을 유지한다.

원래 zookeeper에 저장했는데, zookeeper write overhead 때문에 kafka 내부 토픽으로 관리 (__consumer_offsets?)
consumer group 단위로 해당 파티션을 어디까지 읽었는지 저장
https://soft.plusblog.co.kr/29
