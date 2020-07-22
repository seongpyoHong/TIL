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
