### Elasticsearch Indexing Performance Tuning

인턴 과제를 진행하며 Elasticsearch Indexing 성능을 향상시키기 위해 다음과 같은 설정을 적용하였다.

1. `text` => `keyword`
   
   `text` 필드는 analyzer를 통해 역색인 과정을 거치지만 `keyword` 필드는 데이터 그대로 저장되기 떄문에 인덱싱 속도가 빠르다.

2. `refresh_interval` 
   
   Elasticsearch에서 refresh는 인덱스의 데이터를 새로고침하는 과정으로 기본값은 `1`초이다. 이를 통해 근실시간 검색을 지원하지만 refresh에 비용이 분산된다. 
   
   진행하는 프로젝트는 검색 지연이 1분까지 허용되기 때문에 `refresh_intervl`을 `30`초로 설정하여 인덱스를 새로고침 하는데 발생하는 비용을 줄여 인덱싱 속도를 향상시킬 수 있었다.

3. `replica`
   
   Elasticsearch의 replica shard도 내부적으로 루씬을 가지고 있으며, primary shard와 동일한 검색 결과를 보장하기 위해 동일한 segment 생성 과정을 거쳐야 한다. 따라서 replica 수가 증가할 수록 생성해야하는 segment 수도 증가하여 인덱싱 성능이 하락한다.

   진행 프로젝트에서는 Elasticsearch 앞단에 Kafka를 두어 메세지 손실이 발생할 경우 Re-Consuming을 통해 복구가 가능했기 때문에 `replica`를 `0`으로 줄여 인덱싱 성능을 확보했다. 
   
4. `translog` 

   Elasticsearch는 고가용성을 지원하기 위해 `translog`라는 파일을 유지한다. Shard의 변경사항이 생길 경우 translog 파일에 먼저 해당 내역을 기록한 후, 내부에 존재하는 루씬 인덱스의 인메모리 버퍼에 전달한다. `refresh_interval` 추기마다 인메모리 버퍼의 내용을 처리하는 refresh 작업이 수행되며 이를 통해 새롭게 변경된 세그먼트의 내용이 검색에 반영된다. translog 파일의 내용은 지워지지 않고 유지되기 때문에 Elasticsearch의 장애 복구에도 사용된다. 
   translog의 내용을 계속해서 보관하게되면 용량이 증가하므로, 특정 주기(Elasticsearch의 Flush)가 발생하면 그동안 기록되던 translog의 내용이 삭제된다. 
   
   아래의 2가지 translog 관련 설정으로 가용성을 포기하는 조건으로 인덱싱 성능을 높일 수 있다.
   1. `index.translog.durability=async`
      기본적으로 이 설정은 `request`로 되어있어 매 요청마다 translog 내용을 비우는 commit 과정이 매 요청마다방생한다. `async`로 설정을 변경하여 `sync_interval` 마다 commit이 발생할 수 있도록 하여 인덱싱 성능을 향상시킬 수 있다.
   2. `index.translog.flush_threshold.size`
      기본적으로 index.translog.flush_threshold_size는는 512MB로 설정되어 512MB가 되면 flush된다. index.translog.flush_threshold_size를 늘리면 노드가 트랜스로그 작업을 실행하는 빈도가 낮아지게 되어 flush에 사용되는 리소스 양을 줄일 수 있고 이로 인해 인덱싱 성능이 개선된다. 또한, 플러시 임계값 크기를 늘리면 Elasticsearch 클러스터가 여러 개의 작은 세그먼트 대신 적은 수의 큰 세그먼트도 생성한다. 큰 세그먼트는 병합 빈도가 낮고 이로 인해 더 많은 리소스를 인덱싱에 사용할 수 있다.
