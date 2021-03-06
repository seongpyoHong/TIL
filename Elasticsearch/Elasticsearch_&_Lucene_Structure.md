## Elasticsearch 내부 구조

**[엘라스틱서치 실무 가이드]** 책을 통해 Elasticsearch의 내부 구조 및 Lucene의 구조에 대해 알아보자.



### Segment

하나의 루씬 Index는 다수의 Segment로 이루어져 있다. 루씬이 검색 요청을 받으면 다수의 작은 Segment 조각들이 각자 검색 결과를 만들고 이를 하나로 합쳐서 응답하는 방식으로 동작한다.  Segment에는 색인된 데이터가  역색인 구조로 저장되어 있다. 

- 색인 작업 요청이 들어오면 `IndexWriter`에 의해 색인 작업이 이루어지고 하나의 새로운 세그먼트가 생성된다. 

- 검색 작업 요청이 들어오면 `IndexSearcher`가 Commit Point라는 자료구조를 이용해 모든 Segment를 읽어 검색 결과를 제공한다.



#### Segment 불변성

루씬에서 Segment는 다음과 같은 장점을 위해 수정을 허용하지 않는 불변성의 특성을 가진다. 

1. **동시성 문제를 회피할 수 있다.**

   불변성이 보장된다면 동시성 문제를 회피하기 위한 다른 방법없이 간단하게 회피 가능

2. **시스템 캐시를 적극적으로 활용할 수 있다.**

   불변성이 보장되지 않는 경우, 데이터가 변경될 때마다 시스템 캐시를 삭제하고 다시 생성해야 하므로 성능 측명에서 비용이 큰 작업이 발생한다. 하지만 불변성이 보장됨으로 시스템 캐시가 한 번 생성되며 일정 시간 동안 그대로 유지되어 사용 가능할 수 있다.

3. **눞은 캐시 적중률을 유지할 수 있다.**

   시스템 캐시의 적재되어 있는 데이터들의 수명이 길어지기 때문에 적중률이 높아진다. 이는 검색시 메모리에서 읽어올 수 있는 확률이 높다는 것을 의미한다.

4. **리소스를 절감할 수 있다.**

   Segment에 수정을 허용하게 된다면 일부분이 변경되더라도 전체를 대상으로 역색인을 생성하게 된다. 역색인을 만드는 과정은 리소스를 많이 소모하는 과정이기 때문에 불변성을 통해 리소스를 절감할 수 있다.



##### Segment 삭제 & 수정

- 삭제

  모든 문서에 존재하는 삭제 여부를 표시하는 비트 배열에 표시

  > 검색 시 비트 배열에 설정된 삭제 여부 값을 항상 먼저 판단하기 때문애 불변성을 훼손하지 않고도 빠르게 검색 대상에서 제외할 수 있다. 

- 수정

  Segment의 불변성을 유지하기 위해 해당 데이터를 삭제한 후, 다시 추가하는 방식으로 동작한다. 

---

### Flush & Commit & Merge

#### Flush

Lucene은 효율적인 색인 작업을 위해 내부적으로 일정 크기의 버퍼(`In-Memory Buffer`) 를 가지고 있다. 

- 만약 Lucene 내부에 버퍼가 없다면 데이터가 들어올 떄마다 동기적으로 작업을 수행해야 하기 때문에 데이터를 유실하지 않기위해 매번 세그먼트를 만들어야 한다. 또한, 대량의 데이터가 빠르게 요청될 경우 지연이 발생하여 서비스 장애를 초래할 수 있다.

Lucene에 색인 작업이 요청되면 전달된 데이터는 인메모리 버퍼에 순서대로 쌓이며 내부 버퍼에 일정 크기 이상의 데이터가 쌓이거나 일정 시간이 지난 후 버퍼에 쌓인 데이터를 한꺼번에 처리한다. 한꺼번에 처리된 데이터를 Segment 형태로 생성되며 즉시 디스크로 동기화된다. 

>  새로운 Segment가 생성되고 디스크에 동기화하는 과정까지 마무리되어야 검색이 가능해진다.



이 때, 물리적으로 디스크에 동기화하는 과정은 비용이 큰 연산이기 때문에 Lucene은 `fsync` 대신 `write` 방식을 통해 동기화 과정을 수행한다.

> **Write()**
>  쓰기 요청시 시스템 캐시에만 기록되고 리턴되며, 특정한 주기에 따라 물리적인 디스크로 저장된다. 이로 인해 빠른 처리가 가능하지만 시스템이 비정상 종료될 경우 데이터 유실이 발생할 수 있다.



위에서 설명한 과정을 `Flush` 라고 한다.



#### Commit

Flush 과정에서 `write` 함수를 통해 디스크에 동기화했기 때문에 디스크에 기록되는 것이 100% 보장되지 않는다. 따라서 특정 주기마다 디스크에 직접 기록하는 `fsync` 함수를 실행해야 한다. 이러한 과정을 `Commit` 이라고 한다.



#### Merge

불변성으로 인해 늘어난 다수의 Segment를 합치는 과정을 의미한다. Merge 작업을 통해 다음과 같은 장점을 취할 수 있다.

1. 검색 성능 향상

   검색 요청이 들어오면 Lucene 내부에 존재하는 모든 세그먼트를 검색해야 하는데, 각 Segment는 순차적으로 검색되므로 개수가 줄어들게 되면 검색 횟수도 줄어든다.

2. 저장 용량 감소

   삭제 표시가 된 비트의 경우, Merge 작업 전에는 디스크에 물리적으로 남아있게 된다. Merge 작업을 수행하게 되면 디스크에서 삭제되어 저장 공간을 절약할 수 있다.

Merge 작업은 Commit 작업을 동반하기 때문에 적절한 주기마다 수행하는 것이 좋다.

---

### Elasticsearch vs Lucene

 Elasticsearch는 Lucene의 Flush, Commit, Merge 작업을 그대로 사용하지 않고 고가용성에 적합하도록 개선 및 확장해서 사용한다.

| Lucene | Elasticsearch |
| :----: | :-----------: |
| Flush  |    Refresh    |
| Commit |     Flush     |
| Merge  | Optimize API  |



#### Refresh

Lucene의 Flush 작업을 Elasticsearch에서는 Refresh라고 부르며, Refresh 주기를 수동으로 조절할 수 있는 API를 제공핟다.



#### Flush

Elasticsearch의 Flush는 Lucene의 Commit 작업을 수행하고 새로운 Translog 작성을 시작한다는 의미이다. 

> **Translog?**
>
> 샤드의 장애 복구를 위해 제공되는 파일로 샤드는 자신에게 일어나는 모든 변경사항을 Translog에 먼저 기록한 후 내부에 존재하는 Lucene을 호출한다. 이 때, Refresh가 일정 주기마다 발생하여 NRT 검색을 지원하지만 실제로 디스크에 기록되기 전이므로 동기화를 위해 Commit을 호출해야한다.
> Commit이 수행되면 변경 사항이 디스크에 기록되고 Translog 파일에서 Commit이 일어난 시점까지의 데이터가 삭제되고 새로운 변경사항들이 기록되기 시작한다.



#### Optimize API (force merged API)

Lucene Merge 작업을 강제로 수행하여 다수의 Segment를 하나의 Segment로 통합한다. 
