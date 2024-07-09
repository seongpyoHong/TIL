#### Reference
https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html

#### Data Stream이란?
하나의 named resource을 사용하며 time-series data를 multiple indice에 추가할 수 있는 기능
- Data Stream을 왜 사용해야할까?
	- data가 timestamp field를 가지고 있거나 자동으로 생성되는 경우
	- 색인이 주로 발생하고 갱신이나 삭제는 자주 발생하지 않는 경우
	- document에 ID가 없는 경우, ID가  first-write-wins인 경우
	=> 대부분의 time series data에 대해 data stream은 잘 맞는다


### Backing Indices
Dats stream은 1개 이상의 자동 생성된 backing indice들이 존재한다.
- data stream은 매칭되는 index template이 존재해야한다.
- 하나의 index template은 여러 multiple data stream에 사용될 수 있다.
- data stream에 사용 중인 index template은 삭제가 불가능하다.
- backing indeice의 이름은 구현 디테일이며 unique하다.


#### Read Requests
읽기, 검색 요청은 stream에 의해 모든 backing indice로 전달된다.

#### Write Request
가장 최근에 생성된 backing index가 write index가 되며, 색인 요청은 해당 index로만 보내게된다.

#### Rollover
색인 및 검색 성능을 만족하기 위해 특정 임계점에 도달하면 Index를 새롭게 생성하여 해당 Index에 다시 생성하는 것
Rollver는 ILM을 통해 자동으로 rollover 되도록 설정하는 것을 권장
- 개인적으로 Rollover, Datastream, ILM의 사용 목적이 헷갈렸는데 정리하면 다음과 같다.
	- Rollover의 목적: 조건을 충족할 때 새로운 Index를 만들기 위함
	- ILM의 목적: 자동으로 Rollover를 수행하기 위함
	- Data Stream의 목적: 최소한의 설정으로 write index을 최신으로 유지하기 위함


##### data stream 없이 write index에 대해 index alias를 사용해야 하는 경우에는 다음과 같은 설정이 필요(e.g update, delete가 필요한 경우)
1. Lifecycle Policy 설정 + Index Template 설정
	- auto rollover를 활성화하기 위해서는 밑의 ILM 설정을 template에 추가해야한다.
		- `index.lifecycle.name` : lifecycle policy 이름
		- `index.lifecycle.rollover_alias` : index alias 이름

```json
PUT _index_template/timeseries_template
{
  "index_patterns": ["timeseries-*"],                 
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "timeseries_policy",      
      "index.lifecycle.rollover_alias": "timeseries"    
    }
  }
}
```


2. Bootstrap Index
	1. Index 생성 및 Index Template에 지정된 Rollover Alias에 대한 write index로 지정해야한다.
```json
PUT timeseries-000001
{
  "aliases": {
    "timeseries": {
      "is_write_index": true
    }
  }
}
```

- rollover 컨디션이 만족하면, `timeseries-000002` 가 생성되고, 새롭게 생성된 Index가 write index가 된다. (기존 1번 인덱스는 read-only가 된다.)

- 

