### Limitation Shard's Number in Node
#### 문제 상황
Elasticsearch를 운영하며 처리량을 높이기 위헤 기존 4개였던 Data Node를 12개로 증가 => 12개의 노드에 맞춰 Shard 수를 12개로 증가
1개의 노드에 1개의 Shard를 할당되어야 하지만, 1개의 노드에 2개 이상의 Shard가 할당되는 현상이 발생

#### 해결 방법
`total_shards_per_node` 를 `1`로 명시하여 1개의 노드에 1개의 shard만 생성되도록 제한   

**Index Template**
```JSON
{
...
  "routing" : {
    "allocation" : {
      "total_shards_per_node" : "1"
    }
  }
...
}
```
